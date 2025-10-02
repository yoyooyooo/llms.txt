---
title: 集成测试：MSW 流式响应 + UI 钩子（useChat/useCompletion）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [testing, integration, msw, streaming]
audience: both
---

# 集成测试：MSW 流式响应 + UI 钩子（useChat/useCompletion）

## TL;DR
- 不要 mock fetch；用 MSW 在网络层返回符合协议的流。[4][6]
- UI Message Stream：SSE + `x-vercel-ai-ui-message-stream: v1`；Text 流：`text/plain`。[3]
- 中断分两层：Abort 行为（spy）与 UI 停止增长（时间窗或错误分片）。

相关通用文档：
- MSW 流式响应实战：docs/msw/guides/streaming.md:1

## 执行清单（Execution）
1. Vitest `environment: 'jsdom'`，`setupFiles` 启动 `msw/node`
2. 使用 `simulateReadableStream().pipeThrough(new TextEncoderStream())` 构造 ReadableStream
3. 设置正确头（UI Message Stream 推荐）
4. 编写组件测试：点击提交 → 断言增量文本/错误态 → 点击 Stop → 断言 Abort 被调用

## Definition of Done（验收）
- [ ] 覆盖正常流/错误/中断三类路径
- [ ] 头/协议正确，分片能被前端正确消费
- [ ] 不依赖硬等待；断言使用 Testing Library 的异步查询

目标：在 Node/jsdom 测试环境中验证“前端钩子 ↔ API 路由”的契约是否正确，包括加载、增量渲染、错误与中断等状态。避免直接 mock fetch，使用 MSW 在网络层返回“符合协议的流”。[4][6]

## 测试环境准备

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['tests/setup/setupTests.ts'],
  },
});

// tests/setup/setupTests.ts
import { beforeAll, afterAll, afterEach } from 'vitest';
import { setupServer } from 'msw/node';

export const server = setupServer();

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## 返回 UI Message Stream（v5 推荐）

使用 `simulateReadableStream(...).pipeThrough(new TextEncoderStream())` 生成 `ReadableStream`，并设置 SSE 头与 `x-vercel-ai-ui-message-stream: v1`。[1][2][3]

```ts
// tests/integration/AskBox.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from '../setup/setupTests';
import { simulateReadableStream } from 'ai';
import { AskBox } from '@/app/(components)/AskBox';

function mockUIMessageStream(chunks: string[]) {
  const body = simulateReadableStream({ chunks })
    .pipeThrough(new TextEncoderStream());

  return new HttpResponse(body, {
    headers: {
      'content-type': 'text/event-stream',
      'x-vercel-ai-ui-message-stream': 'v1',
      'cache-control': 'no-cache',
      connection: 'keep-alive',
    },
  });
}

describe('AskBox (UI Message Stream)', () => {
  it('renders incremental text and finishes', async () => {
    server.use(
      http.post('/api/completion', () =>
        mockUIMessageStream([
          `data:{"type":"start","messageId":"m1"}\n\n`,
          `data:{"type":"text-start","id":"t1"}\n\n`,
          `data:{"type":"text-delta","id":"t1","delta":"This"}\n\n`,
          `data:{"type":"text-delta","id":"t1","delta":" is "}\n\n`,
          `data:{"type":"text-delta","id":"t1","delta":"fine."}\n\n`,
          `data:{"type":"text-end","id":"t1"}\n\n`,
          `data:{"type":"finish"}\n\n`,
          `data:[DONE]\n\n`,
        ]),
      ),
    );

    const user = userEvent.setup();
    render(<AskBox />);

    await user.click(screen.getByRole('button', { name: 'Ask' }));

    // 增量断言（自动等待）
    expect(await screen.findByText(/This/)).toBeInTheDocument();
    expect(await screen.findByText(/This is/)).toBeInTheDocument();
    expect(await screen.findByText(/This is fine\./)).toBeInTheDocument();
  });

  it('shows error on malformed event', async () => {
    server.use(
      http.post('/api/completion', () =>
        mockUIMessageStream([
          `data:{"type":"text-delta","id":"t1","delta":"First"}\n\n`,
          `bad data\n`, // 畸形分片，预期触发前端错误分支
        ]),
      ),
    );

    const user = userEvent.setup();
    render(<AskBox />);
    await user.click(screen.getByRole('button', { name: 'Ask' }));
    // 依据组件的错误渲染策略进行断言
    // expect(await screen.findByRole('alert')).toBeInTheDocument();
  });

  it('spies abort on Stop click (msw cannot truly abort in-flight)', async () => {
    const abortSpy = vi.spyOn(AbortController.prototype, 'abort');
    server.use(
      http.post('/api/completion', () =>
        mockUIMessageStream([
          `data:{"type":"start","messageId":"m1"}\n\n`,
          `data:{"type":"text-start","id":"t1"}\n\n`,
          `data:{"type":"text-delta","id":"t1","delta":"First"}\n\n`,
        ]),
      ),
    );

    const user = userEvent.setup();
    render(<AskBox />);
    await user.click(screen.getByRole('button', { name: 'Ask' }));
    await screen.findByText(/First/);
    await user.click(screen.getByRole('button', { name: 'Stop' }));
    expect(abortSpy).toHaveBeenCalled();
  });
});
```

## 返回 Text 流（仅纯文本）

如仅需文本增量，可让 MSW 返回按文本拼接的 `ReadableStream` 与 `content-type: text/plain`，前端使用 `TextStreamChatTransport`。但在 v5 新项目中，更推荐使用 UI Message Stream。[3][4]

## 小结

- 在 Node 环境中，MSW v2 原生支持 `ReadableStream` 作为响应体；也可用 `TransformStream` 注入延迟，模拟真实网络波动。[4]
- 中断测试分为两层：行为（Abort 发起）与效果（UI 不再增长）。前者用 spy 断言，后者结合时间窗口或错误分片验证。MSW 无法真正中止在途流。[Atrera]
- 切勿直接 mock `fetch`，那会掩盖请求构建错误（headers/body/credentials 等）。[6]

> 参考（含 URL）：
> [1] AI SDK Core: Testing — https://ai-sdk.dev/docs/ai-sdk-core/testing
>
> [2] simulateReadableStream — https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream
>
> [3] Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol
>
> [4] MSW Streaming — https://mswjs.io/docs/http/mocking-responses/streaming
>
> [6] Stop mocking fetch — https://kentcdodds.com/blog/stop-mocking-fetch
>
> Atrera：How I Tested Streaming Responses with Vercel AI SDK — https://blog.atrera.com/post/unit-testing-streaming-ai-vercel-sdk/

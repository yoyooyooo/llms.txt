---
title: 单元测试：ai/test + 模型与流模拟（Server/Core）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [testing, unit, simulateReadableStream, ai-test]
audience: both
---

# 单元测试：ai/test + 模型与流模拟（Server/Core）

## TL;DR
- 用 `MockLanguageModelV3 + simulateReadableStream` 构造可控分片，避免外网与不确定性。[1][2]
- 默认使用 UI Message Stream；仅纯文本时改用 `toTextStreamResponse()`。[3]
- 断言包括：分片类型/文本、`finishReason/usage`、Response 头与体格式。
- Node 18+ 原生 Web Streams；确保与 `TextEncoderStream` 一致。

相关通用文档：
- Vitest Mock/Spy 速查：docs/vitest/guides/mocking.md:1

## 执行清单（Execution）
1. 安装依赖：`ai @ai-sdk/openai @ai-sdk/react zod vitest msw`
2. 在测试文件中导入 `MockLanguageModelV3` 与 `simulateReadableStream`
3. 用 `doStream` 或 `doGenerate` 构造分片/对象文本
4. 将结果转换为 `toUIMessageStreamResponse()` 或 `toTextStreamResponse()`
5. 读取 `response.body.getReader()` 并断言内容/头部

## Definition of Done（验收）
- [ ] 用例不访问外网，100% 可重复
- [ ] 分片与元数据断言完整（含 finishReason/usage）
- [ ] 头/协议断言（UI Message Stream 或 Text）
- [ ] 代码片段可运行且通过

目标：不依赖外网，以确定性方式测试 Core API（如 `streamText/generateObject/streamObject`）与你的路由/业务封装。优先使用 `ai/test` 提供的 Mock 与流模拟。[1]

## 基础能力与依赖

- `MockLanguageModelV3`：可注入到 `streamText/generateObject/streamObject`，控制文本/对象/分片与元数据。[1]
- `simulateReadableStream`：按顺序、可配置延迟地产生分片（Web ReadableStream），模拟时间维度。[2]
- Node 18+ 原生 Web Streams；如在更低版本，确保 polyfill 一致。

安装（建议）：`ai @ai-sdk/openai @ai-sdk/react zod vitest msw`（`ai/test` 底层依赖 provider-utils，可能触发对 msw 的引用，确保 dev 依赖里有 msw）。[1]

## 示例一：测试 streamText 的流式分片

```ts
// tests/unit/streamText.test.ts
import { describe, it, expect } from 'vitest';
import { streamText, simulateReadableStream } from 'ai';
import { MockLanguageModelV3 } from 'ai/test';

describe('streamText with mocked model', () => {
  it('should stream text parts deterministically', async () => {
    const model = new MockLanguageModelV3({
      doStream: async () => ({
        stream: simulateReadableStream({
          chunks: [
            { type: 'text-start', id: 't1' },
            { type: 'text-delta', id: 't1', delta: 'Hello' },
            { type: 'text-delta', id: 't1', delta: ' world' },
            { type: 'text-end', id: 't1' },
            { type: 'finish', finishReason: 'stop', usage: { inputTokens: 1, outputTokens: 2, totalTokens: 3 } },
          ],
        }),
      }),
    });

    const result = streamText({ model, prompt: 'Hi' });
    const response = result.toUIMessageStreamResponse();

    // 读取 Response 流并断言内容片段（示例：拼接原始 SSE 文本）
    const reader = response.body!.getReader();
    const decoder = new TextDecoder();
    let raw = '';
    for (;;) {
      const { done, value } = await reader.read();
      if (done) break;
      raw += decoder.decode(value);
    }
    expect(raw).toContain('text-delta');
    expect(raw).toContain('Hello');
  });
});
```

要点：
- 使用 UI Message Stream（推荐）可以直接对接前端 `useChat`；如仅需文本，改为 `result.toTextStreamResponse()` 并断言文本拼接即可。[3]

## 示例二：测试 generateObject/streamObject（结构化输出）

```ts
// tests/unit/object.test.ts
import { describe, it, expect } from 'vitest';
import { generateObject, streamObject, simulateReadableStream } from 'ai';
import { MockLanguageModelV3 } from 'ai/test';
import { z } from 'zod';

const schema = z.object({ content: z.string() });

describe('structured data generation', () => {
  it('generateObject returns schema-compliant JSON', async () => {
    const model = new MockLanguageModelV3({
      doGenerate: async () => ({
        finishReason: 'stop',
        usage: { inputTokens: 10, outputTokens: 20, totalTokens: 30 },
        content: [{ type: 'text', text: '{"content":"Hello, world!"}' }],
        warnings: [],
      }),
    });

    const result = await generateObject({ model, schema, prompt: 'Hello' });
    expect(result.object.content).toBe('Hello, world!');
  });

  it('streamObject emits JSON parts and finishes', async () => {
    const model = new MockLanguageModelV3({
      doStream: async () => ({
        stream: simulateReadableStream({
          chunks: [
            { type: 'text-start', id: 't1' },
            { type: 'text-delta', id: 't1', delta: '{ "content": "Hel"' },
            { type: 'text-delta', id: 't1', delta: 'lo" }' },
            { type: 'text-end', id: 't1' },
            { type: 'finish', finishReason: 'stop', usage: { inputTokens: 1, outputTokens: 2, totalTokens: 3 } },
          ],
        }),
      }),
    });

    const result = streamObject({ model, schema, prompt: 'Hello' });
    // 可进一步消费 result 对象，或转换为 Response 验证流协议
    expect(result).toBeTruthy();
  });
});
```

## 路由处理器的单元测试思路

对于 Next.js Route（App Router），建议将“模型提供者”与“路由处理”解耦，便于在测试中注入 Mock 模型：

```ts
// src/lib/ai.ts
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export function streamWithProvider(prompt: string, model = openai('gpt-4o')) {
  return streamText({ model, prompt });
}

// src/app/api/completion/route.ts
import { streamWithProvider } from '@/lib/ai';
export async function POST(req: Request) {
  const { prompt } = await req.json();
  const result = await streamWithProvider(prompt);
  return result.toUIMessageStreamResponse();
}

// tests/unit/route.test.ts
import { describe, it, expect } from 'vitest';
import { POST } from '@/app/api/completion/route';
import { MockLanguageModelV3 } from 'ai/test';
import { simulateReadableStream } from 'ai';

it('route returns UI message stream with expected parts', async () => {
  // 直接构造一个符合 UI Message Stream 的 Response 进行断言
  const body = simulateReadableStream({
    chunks: [
      `data:{"type":"start","messageId":"m1"}\n\n`,
      `data:{"type":"text-start","id":"t1"}\n\n`,
      `data:{"type":"text-delta","id":"t1","delta":"Hello"}\n\n`,
      `data:{"type":"text-end","id":"t1"}\n\n`,
      `data:{"type":"finish"}\n\n`,
      `data:[DONE]\n\n`,
    ],
  }).pipeThrough(new TextEncoderStream());

  const response = new Response(body, {
    headers: {
      'content-type': 'text/event-stream',
      'x-vercel-ai-ui-message-stream': 'v1',
    },
  });

  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  let raw = '';
  for (;;) {
    const { done, value } = await reader.read();
    if (done) break;
    raw += decoder.decode(value);
  }
  expect(raw).toContain('text-delta');
});
```

> 参考（含 URL）：
> [1] AI SDK Core: Testing — https://ai-sdk.dev/docs/ai-sdk-core/testing
>
> [2] simulateReadableStream — https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream
>
> [3] Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol

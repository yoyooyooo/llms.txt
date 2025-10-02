---
title: 流式响应：ReadableStream 与 SSE（MSW）
tool: msw
status: stable
last_verified: 2025-10-02
tags: [streaming, readable-stream, sse]
audience: both
---

# 流式响应：ReadableStream 与 SSE（MSW）

## TL;DR
- 使用 `ReadableStream` 作为响应体，`TextEncoder`/`TextEncoderStream` 编码分片。[msw-stream]
- SSE 响应需设置 `content-type: text/event-stream`，分片形如 `data: {...}\n\n`。
- 可使用 `TransformStream` 注入网络延迟，模拟真实环境。

## 执行清单（Execution）
1. 在 Vitest/JSDOM 环境的 `setupFiles` 中启动 `msw/node`。
2. 在测试中注册 handler：返回 `new HttpResponse(body, { headers })`。
3. body 可由 `ReadableStream` 构成，SSE 需正确拼接 `data: ...\n\n`。
4. 对于 Vercel AI SDK v5 的 UI Message Stream，增加 `x-vercel-ai-ui-message-stream: v1`。

## 最小示例（SSE JSON 分片）
```ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.post('/api/stream', () => {
    const body = new ReadableStream<Uint8Array>({
      start(controller) {
        const enc = new TextEncoder();
        controller.enqueue(enc.encode('data:{"type":"start"}\n\n'));
        controller.enqueue(enc.encode('data:{"type":"text-delta","delta":"Hello"}\n\n'));
        controller.enqueue(enc.encode('data:{"type":"finish"}\n\n'));
        controller.enqueue(enc.encode('data:[DONE]\n\n'));
        controller.close();
      },
    });
    return new HttpResponse(body, {
      headers: {
        'content-type': 'text/event-stream',
        'x-vercel-ai-ui-message-stream': 'v1',
      },
    });
  }),
];
```

## 使用 TransformStream 注入延迟
```ts
const latency = new TransformStream<Uint8Array, Uint8Array>({
  async transform(chunk, controller) {
    await new Promise(r => setTimeout(r, 100));
    controller.enqueue(chunk);
  },
});

return new HttpResponse(originalStream.pipeThrough(latency), originalResponse);
```

## Definition of Done（验收）
- [ ] ReadableStream 正常返回，客户端能增量消费
- [ ] SSE 分片格式正确（`data: ...\n\n`）
- [ ] 头部设置正确（UI Message Stream 时包含 `x-vercel-ai-ui-message-stream: v1`）

## 参考（URL）
- MSW Streaming — https://mswjs.io/docs/http/mocking-responses/streaming
- Mock SSE with MSW（示例） — https://alexocallaghan.com/mock-sse-with-msw
- MDN Streams API — https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Using_readable_streams


---
title: 流协议详解：Text vs UI Message Stream（v5 推荐）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [protocol, streaming, sse]
audience: both
---

# 流协议详解：Text vs UI Message Stream（v5 推荐）

## TL;DR
- 首选 UI Message Stream（SSE JSON 分片）配合 `useChat`；仅纯文本再用 Text 流。[3]
- UI Message Stream 关键头：`content-type: text/event-stream`、`x-vercel-ai-ui-message-stream: v1`。
- Text 流后端用 `toTextStreamResponse()`；前端用 `TextStreamChatTransport`。
- 测试中用 `simulateReadableStream` 产出规范分片，避免解析错误。[2]

## 执行清单（Execution）
1. 选择协议（优先 UI Message Stream）
2. 后端：`toUIMessageStreamResponse()` 或 `toTextStreamResponse()`
3. 前端：默认 `useChat`（UI Message Stream）或 `TextStreamChatTransport`（Text）
4. 测试：`simulateReadableStream` 生成分片，设置正确头

AI SDK v5 提供两类前后端可协同的流式协议。选择正确协议并在 Mock/测试中保持一致，是稳定测试的基础。[3]

## Text 流（纯文本增量）

- 特点：每个分片是纯文本，前端直接拼接得到最终文案。
- 适用：仅需文本；不需要工具调用、推理或自定义数据。
- 后端返回：`result.toTextStreamResponse()`。
- 前端消费：`TextStreamChatTransport` 或设置 `streamProtocol: 'text'`。

示例（后端）：

```ts
import { streamText, UIMessage, convertToModelMessages } from 'ai';
import { openai } from '@ai-sdk/openai';

export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();
  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
  });
  return result.toTextStreamResponse();
}
```

示例（前端）：

```tsx
'use client';
import { useChat } from '@ai-sdk/react';
import { TextStreamChatTransport } from 'ai';

export default function Chat() {
  const { messages, sendMessage } = useChat({
    transport: new TextStreamChatTransport({ api: '/api/chat' }),
  });
  // ...渲染 messages
}
```

## UI Message Stream（v5 推荐）

- 特点：SSE JSON 分片，支持 `text-*`、`reasoning-*`、`tool-*`、`data-*` 等多种事件，利于复杂 UI。[3]
- 后端返回：`result.toUIMessageStreamResponse()`；需设置头：
  - `content-type: text/event-stream`
  - `x-vercel-ai-ui-message-stream: v1`
- 前端消费：`useChat`（默认 transport 即可）。

SSE 事件示例（部分）：

```txt
data: {"type":"start","messageId":"msg-123"}

data: {"type":"text-start","id":"text-1"}

data: {"type":"text-delta","id":"text-1","delta":"Hello"}

data: {"type":"text-end","id":"text-1"}

data: {"type":"finish"}

data: [DONE]
```

服务端（最小可运行返回）：

```ts
import { streamText, simulateReadableStream } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(req: Request) {
  // 真实场景建议直接 merge result.toUIMessageStream()；此处演示自定义 SSE
  const body = simulateReadableStream({
    chunks: [
      `data:{"type":"start","messageId":"msg-1"}\n\n`,
      `data:{"type":"text-start","id":"t1"}\n\n`,
      `data:{"type":"text-delta","id":"t1","delta":"Hello"}\n\n`,
      `data:{"type":"text-end","id":"t1"}\n\n`,
      `data:{"type":"finish"}\n\n`,
      `data:[DONE]\n\n`,
    ],
  }).pipeThrough(new TextEncoderStream());

  return new Response(body, {
    headers: {
      'content-type': 'text/event-stream',
      'x-vercel-ai-ui-message-stream': 'v1',
      'cache-control': 'no-cache',
      connection: 'keep-alive',
    },
  });
}
```

## 遇到“0:\"...\"”如何处理

- 现象：`0:"Je" 0:" suis" ...` 这类输出来自旧数据流协议的原始片段。[8]
- 方案：
  1) 采用 Core `streamText` + `toTextStreamResponse()` 使用文本协议；
  2) 或迁移至 v5 UI Message Stream（推荐）。[8][9]

> 参考（含 URL）：
> [2] simulateReadableStream — https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream
>
> [3] Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol
>
> [8] Troubleshooting 0:"..." — https://ai-sdk.dev/docs/troubleshooting/strange-stream-output
>
> [9] v5 迁移 — https://ai-sdk.dev/docs/migration-guides/migration-guide-5-0

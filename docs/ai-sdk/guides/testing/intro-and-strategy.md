---
title: 简介与测试策略（测试金字塔）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [testing, strategy, pyramid]
audience: both
---

# 简介与测试策略（测试金字塔）

## TL;DR
- 单测用 `ai/test`，集成用 MSW，E2E 用 Playwright（web-first）。
- 协议优先：优先 UI Message Stream；仅纯文本时使用 Text 流。
- Mock 决策：避免 mock fetch，选择模型 Mock 或网络层 Mock。

## 执行清单（Execution）
1. 统一协议（首选 UI Message Stream）
2. 单测：`MockLanguageModelV3 + simulateReadableStream`
3. 集成：MSW 返回 `ReadableStream` 与正确头
4. E2E：`AI_MOCK=1` 切 Mock，使用自动等待断言

## Definition of Done（验收）
- [ ] 三层测试就位，覆盖关键路径
- [ ] 协议/头一致，前后端同构
- [ ] 不使用直接 mock fetch

随着 LLM 与 Vercel AI SDK 的普及，测试对象从“确定性返回值”扩展到“非确定性、流式增量、含工具/数据分片”的复杂场景。为保证质量与迭代效率，建议沿用“测试金字塔”，并据 AI SDK 的协议与工具特性进行调整：

- 单元测试（占比最大）
  - 面向 Server/Core 逻辑（如 `streamText/generateObject`、路由结果封装），使用 `ai/test` 的 Mock 模型 + `simulateReadableStream`，确保确定性与速度。[1][2]
- 集成测试（契约验证）
  - 前后端接缝（`useChat`/`useCompletion` ↔ API 路由）的验证，使用 MSW 在网络层返回“符合协议的流”（Text 或 UI Message Stream），覆盖加载、增量渲染、错误与中断。[3][4][6]
- 端到端（E2E，关键路径）
  - 真实浏览器 + 真实网络（但服务端切到 Mock），使用 Playwright 的 web-first 断言自动等待流式文本达成预期，避免硬等待带来的不稳定。[5]

## 为什么“协议优先”

- AI SDK v5 推荐 UI Message Stream（SSE JSON 分片），支持文本、推理、工具与自定义数据；仅当你只需纯文本增量时选择 Text 流。[3]
- 协议与头部必须前后端一致：
  - UI Message Stream：`result.toUIMessageStreamResponse()`，`content-type: text/event-stream`，`x-vercel-ai-ui-message-stream: v1`。[1][3]
  - Text：`result.toTextStreamResponse()`。[3]
- 如遇“0:\"...\"”输出，这是旧数据流协议的原始片段，请参照排错与迁移文档，优先升级到 UI Message Stream 或改用 Text 流。[8][9]

## Mock 策略路线图

- 不推荐：直接 mock `fetch`（与实现强耦合，无法发现请求头/体错误）。用 MSW 提升保真度与复用性。[6]
- 单测优先 `ai/test`：在模型接口层 Mock，避免外网与 flakiness，保障确定性回归。[1]
- 集成用 MSW：网络层拦截+返回协议化流，更接近真实运行栈。[4]
- E2E 以“真实浏览器 + Mock 服务端”组合：既验证渲染体验，又避免外部不确定性。[5]

## 版本要点（v4 → v5）

- v5 引入 UI Message Stream，统一了流式协议，推荐后端返回 `toUIMessageStreamResponse()`，前端默认契合 `useChat`。[3][10]
- v5 UI 消息结构采用 `UIMessage.parts`（替代 v4 的 `content`），工具与推理均作为分片。[9][10]
- 旧 Data Stream 的“0:\"...\"”现象见排错：如需纯文本，请改用 `toTextStreamResponse()`；更推荐升级至 UI Message Stream。[8][3]

> 参考（含 URL）：
> [1] AI SDK Core: Testing — https://ai-sdk.dev/docs/ai-sdk-core/testing
>
> [2] simulateReadableStream — https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream
>
> [3] Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol
>
> [4] MSW Streaming — https://mswjs.io/docs/http/mocking-responses/streaming
>
> [5] Playwright Assertions — https://playwright.dev/docs/test-assertions
>
> [6] Stop mocking fetch — https://kentcdodds.com/blog/stop-mocking-fetch
>
> [8] Troubleshooting 0:"..." — https://ai-sdk.dev/docs/troubleshooting/strange-stream-output
>
> [9] v5 迁移 — https://ai-sdk.dev/docs/migration-guides/migration-guide-5-0
>
> [10] useChat — https://ai-sdk.dev/docs/reference/ai-sdk-ui/use-chat

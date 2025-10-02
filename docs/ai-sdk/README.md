---
title: Vercel AI SDK 测试实践总览（Next.js + TypeScript）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [testing, streaming, msw, playwright, vitest]
audience: both
---

# Vercel AI SDK 测试实践总览（Next.js + TypeScript）

> 适配：AI SDK v5 · Next.js App Router · TypeScript · Vitest · MSW · Playwright

本套文档围绕“如何为基于 Vercel AI SDK 的应用建立高质量、可持续的测试体系”，从协议到工具、从单测到 E2E，提供可直接落地的最佳实践与示例代码。

- 为什么要专项关注 AI 流式测试：LLM 非确定性 + 流式 UI 带来的断言与时序难题。
- 协议先行：区分 Text 流与 UI Message Stream，后端返回与前端消费必须匹配。[1][3]
- 测试金字塔：
  - 单元测试（Server/Core）：优先 `ai/test` 的 `MockLanguageModelV3 + simulateReadableStream`，可控、确定、不出网。[1][2]
  - 集成测试（UI+API）：用 MSW 在网络层返回“符合协议的流”，验证钩子与 UI 行为；避免直接 mock fetch。[4][6]
  - 端到端（E2E）：Playwright 的 web-first 断言天然适配流式 UI，配合服务端 Mock 切换，稳定验证增量渲染与中断。[5]
- 版本要点：AI SDK v5 推荐 UI Message Stream（SSE JSON 分片），统一使用 `toUIMessageStreamResponse()`；Text 流仅用于纯文本增量；遇到“0:\"...\"”样式输出参见迁移与排错。[3][8][9]

## 目录
- 01-简介与策略（测试金字塔） → `docs/ai-sdk/guides/testing/intro-and-strategy.md`
- 02-流协议详解（Text vs UI Message Stream） → `docs/ai-sdk/stream-protocols.md`
- 03-单元测试（ai/test + 模型与流模拟） → `docs/ai-sdk/guides/testing/unit-testing.md`
- 04-集成测试（MSW 流式响应 + UI 钩子） → `docs/ai-sdk/guides/testing/integration-testing.md`
- 05-端到端测试（Playwright + Mock 切换） → `docs/ai-sdk/guides/testing/e2e-testing.md`
- 06-覆盖清单与常见坑 → `docs/ai-sdk/guides/testing/checklist-and-pitfalls.md`
- 参考资料 → `docs/ai-sdk/references.md`

## 快速开始（一页版）
- 后端（UI Message Stream）：
  - `const result = streamText(...); return result.toUIMessageStreamResponse();`。[3]
- 前端（useChat）：
  - v5 默认基于 transport，直接 `useChat()` 或自定义 transport；如要纯文本流，再使用 `TextStreamChatTransport`。[3][10]
- 单测：
  - `MockLanguageModelV3 + simulateReadableStream` 构造分片，直测 `streamText/generateObject`、以及封装的 Response。[1][2]
- 集成：
  - MSW 返回 `ReadableStream` + 正确头（`content-type: text/event-stream`、`x-vercel-ai-ui-message-stream: v1`），UI 测试断言增量文本与错误/中断。[1][4]
- E2E：
  - Playwright 配 `webServer` 启动 Next.js，用 `AI_MOCK=1` 切到服务端 Mock，断言 `toHaveText/toContainText` 自动轮询，无需硬等待。[5]

> 引用：见文末“参考资料”与各章节脚注（均附 URL）。

[1] AI SDK Core: Testing — https://ai-sdk.dev/docs/ai-sdk-core/testing

[2] simulateReadableStream — https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream

[3] AI SDK UI: Stream Protocols（Text / UI Message Stream） — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol

[4] MSW: Streaming — https://mswjs.io/docs/http/mocking-responses/streaming

[5] Playwright: Assertions（web-first 断言） — https://playwright.dev/docs/test-assertions

[6] Kent C. Dodds: Stop mocking fetch — https://kentcdodds.com/blog/stop-mocking-fetch

[8] Troubleshooting: stream output contains 0:"..." — https://ai-sdk.dev/docs/troubleshooting/strange-stream-output

[9] Migration: AI SDK 4 → 5 — https://ai-sdk.dev/docs/migration-guides/migration-guide-5-0

[10] AI SDK UI: useChat（v5） — https://ai-sdk.dev/docs/reference/ai-sdk-ui/use-chat

---
title: AI_GUIDE（AI 代理专用 · Vercel AI SDK 测试）
tool: ai-sdk
status: stable
last_verified: 2025-10-02
audience: agent
tags: [agent, checklist]
---

# AI_GUIDE（AI 代理专用 · Vercel AI SDK 测试）

目标：在 Next.js + TypeScript 项目中，为使用 Vercel AI SDK（v5）的流式能力，快速新增/修改接口与测试，保持可重复与稳定。

## Must / Forbid（硬约束）
- Must：后端优先使用 UI Message Stream，返回 `result.toUIMessageStreamResponse()`。
- Must：前端默认使用 `useChat`（v5 transport 架构）。
- Must：单测用 `ai/test`（`MockLanguageModelV3` + `simulateReadableStream`）。
- Must：集成用 MSW（网络层拦截，返回协议化流）。
- Must：E2E 用 Playwright（web-first 断言，避免硬等待）。
- Forbid：直接 mock `fetch`（会掩盖请求头/体错误）。

## 决策树（简版）
- 需要结构化/工具/推理等丰富分片 → UI Message Stream（推荐）
- 仅纯文本增量 → Text 流（`toTextStreamResponse()` + `TextStreamChatTransport`）
- 只测核心逻辑 → `ai/test`（不出网）
- 验证 UI ↔ API 契约 → MSW（ReadableStream + 正确头）
- 验证用户全路径 → Playwright（`AI_MOCK=1` 切服务端 Mock）

## 操作步骤（黄金路径）
1) 新增路由（UI Message Stream）
- 在路由中使用 `streamText`，返回 `result.toUIMessageStreamResponse()`。
- 参见：`docs/ai-sdk/stream-protocols.md`

2) 编写单测（Core）
- 使用 `MockLanguageModelV3` + `simulateReadableStream` 构造分片；
- 断言 Response 流中包含期望事件（如 `text-delta`）。
- 参见：`docs/ai-sdk/unit-testing.md`

3) 集成测试（UI + API）
- 在 Vitest `setupFiles` 中启动 `msw/node`；
- 用 `simulateReadableStream(...).pipeThrough(new TextEncoderStream())` 生成 `ReadableStream`；
- 设置头：`content-type: text/event-stream`，`x-vercel-ai-ui-message-stream: v1`；
- 断言 UI 的增量文本、错误与中断（Stop 按钮 → spy Abort）。
- 参见：`docs/ai-sdk/integration-testing.md`

4) 端到端（E2E）
- `playwright.config.ts` 配 `webServer`；命令里加 `AI_MOCK=1`；
- 断言用 `toHaveText/toContainText`（自动等待），避免 `waitForTimeout`。
- 参见：`docs/ai-sdk/e2e-testing.md`

## Definition of Done（勾选）
- [ ] 后端路由返回 UI Message Stream（或明确选择 Text 流并说明理由）。
- [ ] 单测覆盖核心逻辑，不出网，可重复。
- [ ] 集成测试覆盖：正常流、错误分片/500、中断（Abort spy）。
- [ ] E2E 覆盖主要用户路径，断言不使用硬等待。
- [ ] 文档中附上可点击 URL 的权威参考。

## 关键文件锚点
- `docs/ai-sdk/README.md`
- `docs/ai-sdk/stream-protocols.md`
- `docs/ai-sdk/guides/testing/unit-testing.md`
- `docs/ai-sdk/guides/testing/integration-testing.md`
- `docs/ai-sdk/guides/testing/e2e-testing.md`
- `docs/ai-sdk/guides/testing/checklist-and-pitfalls.md`

## 参考链接（URL）
- Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol
- Core Testing — https://ai-sdk.dev/docs/ai-sdk-core/testing
- simulateReadableStream — https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream
- MSW Streaming — https://mswjs.io/docs/http/mocking-responses/streaming
- Playwright Assertions — https://playwright.dev/docs/test-assertions
- Stop mocking fetch — https://kentcdodds.com/blog/stop-mocking-fetch

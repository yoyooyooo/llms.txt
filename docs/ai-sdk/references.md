---
title: 参考资料（权威链接）
tool: ai-sdk
status: stable
last_verified: 2025-10-02
tags: [references]
audience: both
---

# 参考资料（权威链接）

为保证内容准确，本套文档重点参考以下权威资料：

1) AI SDK Core: Testing（包含 `ai/test`、Mock 模型与流模拟示例）
- https://ai-sdk.dev/docs/ai-sdk-core/testing

2) AI SDK Core: simulateReadableStream（按序/延迟模拟 ReadableStream）
- https://ai-sdk.dev/docs/reference/ai-sdk-core/simulate-readable-stream

3) AI SDK UI: Stream Protocols（Text / UI Message Stream、SSE 事件、必需头）
- https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol

4) MSW: Streaming（返回 ReadableStream 作为响应体、TransformStream 注入延迟）
- https://mswjs.io/docs/http/mocking-responses/streaming

5) Playwright: Assertions（web-first 断言、自动轮询）
- https://playwright.dev/docs/test-assertions

6) Kent C. Dodds: Stop mocking fetch（推荐使用 MSW 替代直接 mock fetch）
- https://kentcdodds.com/blog/stop-mocking-fetch

7) Vitest Mock 指南（`vi.mock/vi.fn/vi.spyOn` 与常见陷阱）
- https://vitest.dev/guide/mocking

8) Troubleshooting：useChat/useCompletion 输出包含 0:"..."（数据流原始片段）
- https://ai-sdk.dev/docs/troubleshooting/strange-stream-output

9) 迁移指南：AI SDK 4 → 5（消息结构、协议与工具更新）
- https://ai-sdk.dev/docs/migration-guides/migration-guide-5-0

10) AI SDK UI: useChat（v5 transport 架构与 API）
- https://ai-sdk.dev/docs/reference/ai-sdk-ui/use-chat

补充参考（社区与生态）：
- Atrera：How I Tested Streaming Responses with Vercel AI SDK（流式单测、中断测试经验）
  - https://blog.atrera.com/post/unit-testing-streaming-ai-vercel-sdk/

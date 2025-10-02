---
title: 覆盖清单与常见坑（排错指南）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [testing, checklist, troubleshooting]
audience: both
---

# 覆盖清单与常见坑（排错指南）

## TL;DR
- 四象限覆盖：正常/错误/中断/部分完成。
- 0:"…" → 老数据流：用 Text 流或升级 UI Message Stream。[8][3]
- MSW 不能真中止：分离 Abort 行为与 UI 停止增长的双层断言。

## 执行清单（Execution）
1. 单测覆盖 `streamText`/对象流与路由封装
2. 集成覆盖 UI Loading/错误/Stop
3. E2E 覆盖完整用户路径
4. 检查协议与头一致性（UI Message Stream 推荐）

## Definition of Done（验收）
- [ ] 四象限至少各 1 条用例
- [ ] 协议/头/分片解析无误
- [ ] 关键报错有清晰定位→修复路径

## 覆盖清单（建议最少集合）

- 单测（Core/路由）：
  - ✅ `streamText`：文本分片序列与 `finishReason/usage` 的断言。[1]
  - ✅ `generateObject/streamObject`：`zod` schema 满足与错误路径（类型不符）。
  - ✅ 路由返回：`toUIMessageStreamResponse()` 头与 SSE 格式正确（或 `toTextStreamResponse()`）。[3]
- 集成（UI+API）：
  - ✅ 正常流：增量渲染与 Loading 消失。
  - ✅ 错误：畸形分片/HTTP 500 → UI 错误态。
  - ✅ 中断：`AbortController.abort` 被调用；UI 文本不再增长（时间窗或错误分片双保险）。[4]
- E2E：
  - ✅ 真实浏览器看到增量文本出现。
  - ✅ Stop 行为后，文本长度保持或最终完成。
  - ✅ 关键用户路径（如对话主流程）至少 1 条高价值用例。

## 常见坑与修复

1) “0:\"...\"” 原始分片出现在前端
- 背景：v3.0.20 起默认使用数据流协议，看到 `0:"..."` 说明在消费旧格式的原始串。[8]
- 修复：
  - 采用 `streamText(...).toTextStreamResponse()` 使用文本协议；或
  - 升级到 v5 UI Message Stream（`toUIMessageStreamResponse()`），并在前端使用 `useChat` 默认 transport。[3][8][9]

2) `ai/test` 报错找不到 `msw`
- 说明：`ai/test` 通过 provider-utils/test 引入测试工具，部分场景会引用 `msw`。[1]
- 修复：把 `msw` 加入 devDependencies。

3) MSW 无法真正“中止在途流”
- 现象：点击 Stop 后，MSW 仍会把流的剩余分片推送完。
- 对策：分离“行为”与“效果”两层断言：
  - 行为：对 `AbortController.prototype.abort` 做 spy/assert；
  - 效果：在测试里限定时间窗内文本不再增长，或通过错误分片提前终止 UI 消费。[Atrera]

4) Web Streams 兼容性问题
- 现象：`ReadableStream is undefined` 或 decode 错误。
- 对策：使用 Node 18+；避免旧 polyfill 与原生实现冲突；确保 `TextEncoderStream`/`TransformStream` 在当前环境可用。[4]

5) 直接 mock fetch 导致用例脆弱
- 现象：请求头/体错误被忽略（测试仍然通过）。
- 对策：改用 MSW；在网络层验证完整请求/响应链路。[6]

## 版本迁移提示（v4 → v5）

- UI 侧：`Message.content` → `UIMessage.parts`；工具/推理作为分片。[9][10]
- 协议：统一使用 UI Message Stream（推荐），或显式使用 Text；避免混用老 Data Stream 片段与新 UI 分片。
- 推荐运行 `npx @ai-sdk/codemod upgrade` 辅助迁移。[9]

> 参考（含 URL）：
> [1] AI SDK Core: Testing — https://ai-sdk.dev/docs/ai-sdk-core/testing
>
> [3] Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol
>
> [4] MSW Streaming — https://mswjs.io/docs/http/mocking-responses/streaming
>
> [6] Stop mocking fetch — https://kentcdodds.com/blog/stop-mocking-fetch
>
> [8] Troubleshooting 0:"..." — https://ai-sdk.dev/docs/troubleshooting/strange-stream-output
>
> [9] v5 迁移 — https://ai-sdk.dev/docs/migration-guides/migration-guide-5-0
>
> [10] useChat — https://ai-sdk.dev/docs/reference/ai-sdk-ui/use-chat
>
> Atrera：How I Tested Streaming Responses with Vercel AI SDK — https://blog.atrera.com/post/unit-testing-streaming-ai-vercel-sdk/

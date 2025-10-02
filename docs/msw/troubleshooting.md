---
title: MSW 故障排查
tool: msw
status: stable
last_verified: 2025-10-02
tags: [troubleshooting, streaming]
audience: both
---

# MSW 故障排查

## TL;DR
- Node 环境需支持 Web Streams；优先使用 Node 18+。
- SSE 必须使用 `text/event-stream` 且 `data: ...\n\n` 分片。
- 不要尝试“真正中止”在途流，用行为（Abort spy）与效果（文本不再增长）分离验证。

## 常见问题
- ReadableStream undefined/locked：检查 Node 版本或 polyfill 冲突；避免多次读取同一可锁流。
- SSE 解析失败：确认 `\n\n` 间隔与 `data:` 前缀正确；不要遗漏结尾 `[DONE]`（如协议要求）。
- 需要注入延迟：使用 `TransformStream` 或 `delay()` 模拟网络。

## 参考（URL）
- MSW Streaming — https://mswjs.io/docs/http/mocking-responses/streaming
- MDN Streams API — https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Using_readable_streams


---
title: MSW（Mock Service Worker）实践总览
tool: msw
status: stable
last_verified: 2025-10-02
tags: [msw, mocking, streaming]
audience: both
---

# MSW（Mock Service Worker）实践总览

MSW 通过拦截网络层请求来进行高保真 Mock，既可用于测试，也可用于本地开发联调。相较直接 mock fetch，它能验证完整的请求构建与响应处理链路。

## 导航
- 流式响应（ReadableStream/SSE）：`docs/msw/guides/streaming.md`
- 故障排查：`docs/msw/troubleshooting.md`
- 参考资料：`docs/msw/references.md`
- AI 执行清单：`docs/msw/AI_GUIDE.md`

## 适用场景
- 集成测试中模拟后端接口（含流式返回）
- 本地开发在接口不可用时的替代
- 验证 headers/body/credentials 等请求构建是否正确


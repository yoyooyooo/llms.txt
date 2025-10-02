---
title: AI_GUIDE（AI 代理专用 · MSW）
tool: msw
status: stable
last_verified: 2025-10-02
audience: agent
tags: [agent, checklist]
---

# AI_GUIDE（AI 代理专用 · MSW）

## Must / Forbid
- Must：在 Node/JSDOM 测试中使用 `msw/node` 拦截网络请求
- Must：流式响应用 `ReadableStream`，SSE 用 `text/event-stream` 与 `data: ...\n\n`
- Forbid：直接 mock fetch

## 执行步骤
1) 在 `setupFiles` 启动 `setupServer()`，并在 `beforeAll/afterEach/afterAll` 管理生命周期
2) 在测试中 `server.use(http.get/post(...))` 返回 `HttpResponse(ReadableStream, { headers })`
3) 对于 AI SDK UI Message Stream，设置 `x-vercel-ai-ui-message-stream: v1`

## DoD（验收）
- [ ] 客户端能够增量消费分片
- [ ] SSE 分片格式与头部正确
- [ ] 不使用直接 mock fetch

## 锚点
- `docs/msw/guides/streaming.md`
- `docs/msw/troubleshooting.md`
- `docs/msw/references.md`


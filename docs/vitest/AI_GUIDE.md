---
title: AI_GUIDE（AI 代理专用 · Vitest）
tool: vitest
status: stable
last_verified: 2025-10-02
audience: agent
tags: [agent, checklist]
---

# AI_GUIDE（AI 代理专用 · Vitest）

## Must / Forbid
- Must：根据场景选择 `environment: 'jsdom'|'node'`
- Must：通过 `setupFiles` 启动 MSW（如需网络拦截）
- Forbid：在可用 MSW 的情况下直接 mock fetch

## 执行步骤
1) 配置 `vitest.config.ts`（环境、setupFiles、覆盖率）
2) 使用 `vi.fn/vi.spyOn` 校验调用；`vi.mock` 替换模块
3) 每个用例清理 mocks，避免串测

## DoD
- [ ] 配置文件与运行脚本可用
- [ ] Mock 层级合理，测试可重复
- [ ] 与 MSW/AI SDK 文档的约束一致

## 锚点
- `docs/vitest/guides/mocking.md`
- `docs/vitest/guides/setup.md`
- `docs/vitest/troubleshooting.md`


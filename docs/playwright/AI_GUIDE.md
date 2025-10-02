---
title: AI_GUIDE（AI 代理专用 · Playwright）
tool: playwright
status: stable
last_verified: 2025-10-02
audience: agent
tags: [agent, checklist]
---

# AI_GUIDE（AI 代理专用 · Playwright）

## Must / Forbid
- Must：使用 web-first 断言（`toHaveText/toContainText`）验证流式 UI
- Must：`webServer` 启动 Next.js，按需用 `AI_MOCK=1`
- Forbid：大量使用 `waitForTimeout` 作为主断言手段

## 执行步骤
1) 在 `playwright.config.ts` 配置 `webServer` 与 `baseURL`
2) 用例中触发操作 → 断言 `toContainText('Hello')`
3) Stop 行为：比较文本长度在短时间窗内不再增长

## DoD（验收）
- [ ] 断言均为 web-first 自动等待
- [ ] CI 与本地皆可运行
- [ ] Mock 切换可控

## 锚点
- `docs/playwright/guides/web-first-assertions.md`
- `docs/playwright/guides/web-server.md`
- `docs/playwright/troubleshooting.md`


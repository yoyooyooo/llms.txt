---
title: Playwright 故障排查
tool: playwright
status: stable
last_verified: 2025-10-02
tags: [troubleshooting]
audience: both
---

# Playwright 故障排查

## TL;DR
- 避免基于 `textContent()` 的一次性断言；改用 `toHaveText/toContainText`。
- 谨慎使用 `waitForTimeout`；优先使用自动等待断言或 `expect.poll`。
- 为动态内容提供稳定的 `data-testid`。

## 常见问题
- 间歇失败（Flaky）：断言不等待，或选择器不稳定；改用 web-first 断言与稳定选择器。
- 服务器未就绪：`webServer` 未配置或启动过慢；显式配置 `url` 与命令。

## 参考（URL）
- Playwright Best Practices — https://playwright.dev/docs/best-practices
- Playwright Assertions — https://playwright.dev/docs/test-assertions


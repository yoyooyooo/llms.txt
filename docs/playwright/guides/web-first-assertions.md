---
title: Web-First 断言（流式 UI 的稳定断言）
tool: playwright
status: stable
last_verified: 2025-10-02
tags: [assertions, streaming]
audience: both
---

# Web-First 断言（流式 UI 的稳定断言）

## TL;DR
- 使用 `expect(locator).toHaveText()/toContainText()`，Playwright 自动轮询直到满足或超时。[pw-assert]
- 避免 `waitForTimeout` 硬等待；必要时用 `expect.poll` 表达自定义轮询。
- 给关键元素加稳定选择器（如 `data-testid`）。

## 执行清单（Execution）
1. 在用例中定位 UI 容器（如 `[data-testid="completion"]`）
2. 触发操作（提交/发送），随后断言 `toContainText('Hello')`
3. 对“停止”行为：记录长度 → 点击 Stop → 短时等待 → 再取长度对比不再增长

## 最小示例
```ts
await page.getByRole('button', { name: 'Ask' }).click();
await expect(page.getByTestId('completion')).toContainText('Hello');
const before = await page.getByTestId('completion').innerText();
await page.getByRole('button', { name: 'Stop' }).click();
await page.waitForTimeout(300);
const after = await page.getByTestId('completion').innerText();
expect(after.length).toBeGreaterThanOrEqual(before.length);
```

## Definition of Done（验收）
- [ ] 断言均为 web-first 自动等待
- [ ] 不依赖固定超时（除非对“停止增长”这类对比短窗）
- [ ] 提供稳定选择器

## 参考（URL）
- Playwright Assertions — https://playwright.dev/docs/test-assertions
- Best Practices — https://playwright.dev/docs/best-practices


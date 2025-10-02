---
title: Next.js webServer 集成（Playwright）
tool: playwright
status: stable
last_verified: 2025-10-02
tags: [webserver, nextjs]
audience: both
---

# Next.js webServer 集成（Playwright）

## TL;DR
- 在 `playwright.config.ts` 配置 `webServer.command` 启动 Next.js（可加 `AI_MOCK=1` 切 Mock）。
- 使用 `use.baseURL` 指向本地服务；`reuseExistingServer` 提升本地迭代效率。

## 最小配置示例
```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  use: { baseURL: 'http://127.0.0.1:3000' },
  webServer: {
    command: 'AI_MOCK=1 npm run dev',
    url: 'http://127.0.0.1:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Definition of Done（验收）
- [ ] 本地与 CI 均可启动与运行用例
- [ ] 支持环境变量切换到 Mock
- [ ] 用例使用 `baseURL` 简化 `goto()`

## 参考（URL）
- Playwright Docs — https://playwright.dev/docs/test-assertions


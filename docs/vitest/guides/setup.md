---
title: 测试环境与配置（Vitest）
tool: vitest
status: stable
last_verified: 2025-10-02
tags: [setup, jsdom]
audience: both
---

# 测试环境与配置（Vitest）

## TL;DR
- UI 测试用 `environment: 'jsdom'`；Node 侧用 `environment: 'node'`
- `setupFiles` 启动 MSW；`server: { deps: { inline: [...] } }` 解决某些包 ESM 解析
- 覆盖率：`coverage.reporter: ['text','html']`

## 最小配置
```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['tests/setup/setupTests.ts'],
    coverage: { reporter: ['text', 'html'] },
  },
});
```

## Definition of Done（验收）
- [ ] JSDOM 环境下 UI 测试通过
- [ ] MSW 生命周期管理清晰
- [ ] 覆盖率输出可用

## 参考（URL）
- Vitest 配置 — https://vitest.dev/config/


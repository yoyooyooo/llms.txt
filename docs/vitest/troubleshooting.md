---
title: Vitest 故障排查
tool: vitest
status: stable
last_verified: 2025-10-02
tags: [troubleshooting]
audience: both
---

# Vitest 故障排查

## TL;DR
- 同文件内部函数相互调用无法由外部 mock：考虑拆分/依赖注入
- 计时器/时间相关：使用 `vi.useFakeTimers/vi.setSystemTime`
- ESM/路径别名：通过 `test.alias` 或插件解决

## 参考（URL）
- Vitest Mock 指南 — https://vitest.dev/guide/mocking
- Vitest API（计时器） — https://vitest.dev/api/vi


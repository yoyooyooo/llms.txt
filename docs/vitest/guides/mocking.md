---
title: Mock 与 Spy（Vitest）
tool: vitest
status: stable
last_verified: 2025-10-02
tags: [mock, spy]
audience: both
---

# Mock 与 Spy（Vitest）

## TL;DR
- `vi.fn/vi.spyOn`：函数级替换与调用校验
- `vi.mock`：模块级替换（注意陷阱：同文件内部引用无法被外部 mock 覆盖）[vitest-mock]
- 注意清理：`vi.restoreAllMocks/vi.resetAllMocks`

## 执行清单（Execution）
1. 选择正确层级：函数 spy → vi.spyOn；实现替换 → vi.fn/vi.mock
2. 每个用例后清理 mocks，避免串测
3. 对同文件内部相互调用的函数，考虑解耦或依赖注入

## 最小示例
```ts
import { vi } from 'vitest';
const fn = vi.fn().mockReturnValue('ok');
expect(fn()).toBe('ok');
```

## Definition of Done（验收）
- [ ] mock 层级合理，不与实现强耦合
- [ ] 用例间不串状态
- [ ] 对“不可 mock 场景”做了架构改造或依赖注入

## 参考（URL）
- Vitest Mock 指南 — https://vitest.dev/guide/mocking


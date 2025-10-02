---
title: 端到端测试：Playwright + 服务端 Mock 切换（真实浏览器流）
tool: ai-sdk
versions:
  ai: ">=5.0"
  node: ">=18"
  nextjs: ">=14"
status: stable
last_verified: 2025-10-02
tags: [testing, e2e, playwright, streaming]
audience: both
---

# 端到端测试：Playwright + 服务端 Mock 切换（真实浏览器流）

## TL;DR
- 使用 Playwright web-first 断言（`toHaveText/toContainText` 自动等待），避免硬等待。[5]
- 用 `AI_MOCK=1` 切服务端 Mock；浏览器端体验真实网络与渲染路径。
- 对 Stop 行为：比较文本长度在短时间窗内不再增长。

相关通用文档：
- Web-First 断言：docs/playwright/guides/web-first-assertions.md:1
- Next.js webServer 配置：docs/playwright/guides/web-server.md:1

## 执行清单（Execution）
1. `playwright.config.ts` 配置 `webServer` 启动 Next.js
2. Next.js 路由内根据 `process.env.AI_MOCK` 注入 Mock 模型
3. 用例：访问页面→输入→点击→断言增量文本→点击 Stop→断言长度不再增长

## Definition of Done（验收）
- [ ] 用例不依赖固定超时，断言均自动等待
- [ ] 关键用户路径至少覆盖一次完整流式交互
- [ ] Mock 切换可控（本地与 CI 一致）

目标：在真实浏览器中验证用户路径与流式渲染体验，包括增量文本可见、错误与中断按钮行为。利用 Playwright 的 web-first 断言（自动轮询），避免硬等待与脆弱超时。[5]

## 配置 Playwright 启动 Next.js

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  use: { baseURL: 'http://127.0.0.1:3000', trace: 'on-first-retry' },
  webServer: {
    command: 'AI_MOCK=1 npm run dev', // 用环境变量切换到 Mock 模式
    url: 'http://127.0.0.1:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## 服务端：Mock 切换（建议在 lib 层集中管理）

```ts
// src/lib/ai.ts
import { streamText, simulateReadableStream } from 'ai';
import { openai } from '@ai-sdk/openai';
import { MockLanguageModelV3 } from 'ai/test';

export function streamWithProvider(prompt: string) {
  if (process.env.AI_MOCK === '1') {
    const model = new MockLanguageModelV3({
      doStream: async () => ({
        stream: simulateReadableStream({
          chunks: [
            { type: 'text-start', id: 't1' },
            { type: 'text-delta', id: 't1', delta: 'Hello' },
            { type: 'text-delta', id: 't1', delta: ' world' },
            { type: 'text-end', id: 't1' },
            { type: 'finish', finishReason: 'stop' },
          ],
        }),
      }),
    });
    return streamText({ model, prompt });
  }
  return streamText({ model: openai('gpt-4o'), prompt });
}

// src/app/api/completion/route.ts
import { streamWithProvider } from '@/lib/ai';
export async function POST(req: Request) {
  const { prompt } = await req.json();
  const result = await streamWithProvider(prompt);
  return result.toUIMessageStreamResponse(); // v5 推荐
}
```

## E2E 用例

```ts
// e2e/ask-flow.spec.ts
import { test, expect } from '@playwright/test';

test('user sees incremental response and can stop', async ({ page }) => {
  await page.goto('/');

  await page.getByRole('textbox', { name: /prompt/i }).fill('Hi');
  await page.getByRole('button', { name: 'Ask' }).click();

  // web-first 断言：自动轮询直到包含文本
  await expect(page.getByTestId('completion')).toContainText('Hello');
  await expect(page.getByTestId('completion')).toContainText('Hello world');

  // 中断：点击 Stop 后，文本不再增长
  const before = await page.getByTestId('completion').innerText();
  await page.getByRole('button', { name: 'Stop' }).click();
  await page.waitForTimeout(300);
  const after = await page.getByTestId('completion').innerText();
  expect(after.length).toBeGreaterThanOrEqual(before.length);
});
```

## 最佳实践

- 用环境变量切换 Mock，确保 E2E 稳定、可重放；仍保留真实浏览器的网络/渲染路径。
- 断言以最终状态为主，必要时验证中间 Loading/错误态；优先 `toHaveText/toContainText`，避免 `waitForTimeout`。
- 给关键 DOM 添加 `data-testid`，提高定位与稳定性。

> 参考（含 URL）：
> [3] Stream Protocols — https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol
>
> [5] Playwright Assertions — https://playwright.dev/docs/test-assertions

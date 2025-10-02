# 三方库与 LLMs 文档库

## TL;DR

- 本仓库专注于提炼第三方库/工具的使用方法与 LLMs 实践笔记。
- 所有文档统一放在 `docs/`；每个工具使用 `docs/<slug>/` 独立目录（小写连字符）。
- LLMs 笔记与实验放在 `llms/` 与 `llms.txt:1`。
- 文档统一中文，包含 TL;DR、示例与可点击 URL 的参考链接。

## 快速索引

- AI SDK：`docs/ai-sdk/`
  - 单元测试示例：`docs/ai-sdk/guides/testing/unit-testing.md:1`
  - 集成测试示例：`docs/ai-sdk/guides/testing/integration-testing.md:1`
- MSW：`docs/msw/`
- Playwright：`docs/playwright/`
- Vitest：`docs/vitest/`
- tldraw：`docs/tldraw/`
- LLMs：`llms/`、`llms.txt:1`

## 文档结构约定（简版）

- `docs/<slug>/`
  - `README.md` 概览与导航
  - `guides/` 任务导向的实践（可分主题，例如 `testing/`）
  - `templates/` 可复制代码模板
  - `troubleshooting.md` 常见问题排查
  - `references.md` 参考链接（必须含 URL）
  - 如需：`overview.md` 背景、`checklists/` 执行清单、`AI_GUIDE.md`（可选）

## 写作风格

- 统一中文；短句直达要点，避免冗长背景叙事。
- 每篇尽量包含：TL;DR、执行步骤/示例代码、参考链接（含 URL）。
- 标注运行环境与版本（如 `node>=18`、库版本）以便复现。
- 文件命名使用 `kebab-case`，内链用可点击路径（可加行号），如 `docs/tldraw/guides/shape-prop-migrations.md:1`。

## 新增与贡献

1. 创建目录：`docs/<slug>/`
2. 至少添加：`README.md` 与 `references.md`
3. 视需要补充：`guides/`、`templates/`、`troubleshooting.md`
4. 在相关主题间添加交叉链接（例如 AI SDK 与 MSW 的流式测试）

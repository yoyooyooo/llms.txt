# AGENTS.md（面向人类与 AI 代理的协作约定）

> 本仓库沉淀各类开源工具/三方库的使用方法、API 指南与特殊场景最佳实践。本文定义文档结构、写作规范与对 AI 代理的行为约束，确保“可学、可用、可自动化”。

## 1. 目录与归档规则（必读）

- 文档根目录：`docs/`
- 每个工具/库一个独立目录：`docs/<slug>/`
  - 建议 slug：全小写、连字符，例如 `ai-sdk`、`msw`、`playwright`、`vitest`、`nextjs`。
  - 目录内推荐结构：
    - `README.md`（概览/导航）
    - `overview.md`（可选，背景与术语）
    - `guides/`（任务导向的实践，按主题再分子目录，如 `testing/`）
    - `templates/`（可复制代码模板）
    - `checklists/`（执行与验收清单）
    - `troubleshooting.md`（故障排查）
    - `references.md`（权威链接合集，务必包含 URL）
    - `AI_GUIDE.md`（机器友好的极简执行清单与约束，详见第 4 节）
- 交叉主题文档：优先放在对应工具目录；如属通用能力（如“流式测试通用模式”），放在 `docs/patterns/` 并在各工具目录交叉链接。

## 2. 文档写作规范（人类可读）

- 语言：统一中文。
- 结构：每篇文档尽量包含 TL;DR、执行清单、示例代码、验收标准（Definition of Done）、参考链接（必须含 URL）。
- 风格：简洁直接，代码块可运行；必要时标注运行环境与版本。引用外部资料务必给出可点击 URL（不要只写标题）。
- 文件命名：`kebab-case`，避免空格与中文文件名。
- 内链：引用本仓库文件时，用可点击路径，例如 `docs/ai-sdk/unit-testing.md:1`。

## 3. 版本与元数据（Front‑Matter 建议）

建议在文档开头用列表注明：

- title: 文档标题
- tool: 工具/库名（如 ai-sdk）
- versions: 关键版本（如 ai>=5、node>=18、nextjs>=14）
- status: 草稿/稳定
- last_verified: 最后验证日期（YYYY-MM-DD）
- tags: 关键词（如 testing, streaming, msw, playwright, vitest）
- audience: human | agent | both

## 4. AI 专用文档（AI_GUIDE.md）

- 目的：为 AI 代理提供“短小、严格、机器友好”的执行清单与不可违背约束。
- 每个工具目录下应包含 `AI_GUIDE.md`，内容建议：Must/Forbid、决策树（场景→推荐）、操作步骤、DoD（验收标准）、关键文件锚点、参考链接（URL）。
- 该文档应可被直接注入到 AI 上下文中，无需再阅读长文即可执行任务。

## 5. 对 AI 代理的行为约束

- 修改规范：
  - 用最小改动达成目标；遵循本文件与目标目录下的 `AI_GUIDE.md`。
  - 先解释计划，再执行修改；使用 `apply_patch` 编辑文件。
  - 不新增版权/License 头；不做无关重构；不全局格式化。
- 研究规范：
  - 涉及外部知识时，优先使用 Tavily 搜索与 Fetcher 抓取页面，引用需附 URL。
- 测试与验证：
  - 若文档包含可运行示例，优先添加最小验证脚本或说明如何本地验证。
- 参考与链接：
  - 外链必须可点击 URL；内链使用明确路径（可加行号）。

## 6. 新增一个工具/库文档时的模板流程

1) 创建目录：`docs/<slug>/`
2) 添加：`README.md`、`AI_GUIDE.md`、`references.md`
3) 按主题添加 `guides/`、`templates/`、`troubleshooting.md`（可后续补齐）
4) 在相关文档中添加交叉链接（如 `docs/ai-sdk/` 链接到 `docs/msw/` 的 Streaming）
5) 自查 DoD：是否提供可复制示例、是否有 URL、是否标注版本/环境、是否能被 AI 直接复用。

## 7. 约定与反例

- ✅ 推荐：在测试类文档中提供“Do/Don’t”“Definition of Done”“常见错误→修复映射表”。
- ❌ 避免：只给概念不给示例、只给标题不附 URL、把人类叙事与 AI 指令混在同一长文中。

---

如有大型重构或新增模块，请在 PR 中附上一页“目录结构+导航变化”说明，方便后续检索与 AI 上下文配置。


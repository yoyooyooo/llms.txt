# 新三方库标准化沉淀 · 启动 Prompt（中文模板）

本模板用于在本仓库中为任意三方库/工具创建一套合规的知识沉淀与 AI 执行清单。请替换占位符并直接投喂给代理执行。

---

## 完整版 Prompt（替换占位符后直接使用）

请按本仓库 AGENTS.md 规范，为 <LIB_NAME> 做标准化沉淀。要求：

- 研究范围
  - 目标版本/生态：<LIB_NAME>（<VERSION_CONSTRAINTS>），相关生态：<RELATED_STACK>（如 Node/Next/React/Playwright/Vitest）。
  - 主题列表：<TOPICS>（如 安装配置、核心 API、测试最佳实践、常见坑、性能）。

- 产出物（必须全部给出）
  - 目录：`docs/<SLUG>/`（kebab-case）
    - `README.md`（导航/总览）
    - `guides/` 下按主题拆分（如 `guides/testing/…`）
    - `templates/`（可复制最小示例）
    - `troubleshooting.md`、`references.md`
    - `AI_GUIDE.md`（AI 专用执行清单）
  - 每篇文档必须包含前置元数据（title/tool/versions/status/last_verified/tags/audience），并在文首加入：
    - TL;DR（3–7 条）
    - 执行清单（7 步内）
    - Definition of Done（可勾选）
  - 生成 `llms/<SLUG>-<PRIMARY_TOPIC>.llms.txt`（机器友好、短小、含示例与仓库锚点）
  - 将新清单挂到根 `llms.txt`

- 硬约束
  - 必须使用 Tavily 搜索 + Fetcher 抓取权威资料；所有引用必须附可点击 URL。
  - 代码示例可运行（标注版本/环境），优先给最小可行样例；避免伪代码。
  - 避免直接 mock fetch（如涉及测试，优先 MSW/官方 Mock 工具）；遵循 `AGENTS.md`。

- 交叉链接
  - 如文中涉及 MSW/Playwright/Vitest/Next 等，交叉链接到对应目录（若不存在，可创建骨架）。

- 输出与实施
  - 先给出计划（步骤）与澄清问题（如有），再用 `apply_patch` 写入文件。
  - 完成后汇总变更路径清单。

补充信息：
- SLUG = <SLUG>
- 目标系统版本：Node <NODE_VERSION>；框架 <FRAMEWORK_VERSION>
- 偏好语言：中文
- 验收标准：
  - 文档完整、引用可点击、示例可运行、`AI_GUIDE.md` 与 `llms` 清单齐全、根 `llms.txt` 有索引

请先列“计划（steps）+ 澄清问题（如果有）”，再开始实现。

---

## 轻量快速版 Prompt（最小可用）

为 <LIB_NAME>（<VERSION>）生成最小沉淀：
- 产出：`docs/<SLUG>/README.md`、`AI_GUIDE.md`、`references.md`
- 生成：`llms/<SLUG>-quickstart.llms.txt`，并写入根 `llms.txt` 索引
- 要求：所有参考附 URL；给 1 个最小可运行示例与 DoD；中文
- 调研：使用 Tavily + Fetcher 采信官方文档/博客/指南链接
- 执行：先计划后 `apply_patch`，完成后汇总变更清单

---

## 增量更新版 Prompt（已有目录时）

对 `docs/<SLUG>/` 做增量更新：
- 对齐 AGENTS.md 规范：为所有文档补前置元数据、TL;DR、执行清单、DoD、URL 引用
- 新增 `AI_GUIDE.md` 与 `llms/<SLUG>-<TOPIC>.llms.txt`（如缺失）
- 将跨工具引用改为交叉链接（如 MSW/Playwright）
- 给出差异计划并用 `apply_patch` 执行

---

## 建议的澄清问题（由代理主动追问）
- 目标版本/生态范围（库版本、Node/框架版本）？
- 主要使用场景与优先主题（先做哪些）？
- 期望读者与语言（human/agent/both，中文/英文）？
- 是否允许创建交叉主题目录（如 `docs/msw`）？
- 是否需要 E2E 测试/CI 示例片段？

---

## 验收清单（供你快速核对）
- [ ] 目录结构符合约定（`docs/<slug>/…`）
- [ ] 文档含前置元数据 + TL;DR/执行清单/DoD/URL（四件套）
- [ ] `AI_GUIDE.md` 可直接注入模型上下文执行
- [ ] `llms/<slug>-*.llms.txt` 简短、规则明确、含示例/锚点/URL；根 `llms.txt` 有索引
- [ ] 引用全部是点击即到的 URL；示例可运行（注明环境）

---

## 示例：为 Axios 快速启动（参考）

请为 Axios（^1.x）生成最小沉淀，SLUG=axios：
- 主题：安装与基础用法、拦截器最佳实践、错误处理、测试（MSW）
- 产出：`docs/axios/`（`README.md`、`AI_GUIDE.md`、`references.md`）、`llms/axios-quickstart.llms.txt`、根 `llms.txt` 索引
- 引用必须包含 URL（官方文档、MSW 文档等）
- 示例：拦截器 + 重试 + 失败回退的最小代码；测试用 MSW 拦截网络
- Chinese；DoD 齐全；用 Tavily + Fetcher 做调研并附链接
- 先给计划与澄清，再用 `apply_patch` 写入文件


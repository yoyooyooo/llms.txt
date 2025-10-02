title: 参考资料（权威链接）
tool: tldraw
versions: [tldraw>=4]
status: stable
last_verified: 2025-10-02
tags: [references]
audience: both

# 参考资料（权威链接）

- 官方站点与仓库
  - tldraw 官网与文档：https://tldraw.dev/
  - GitHub（源码与示例）：https://github.com/tldraw/tldraw
  - NPM（`tldraw` 包）：https://www.npmjs.com/package/tldraw

- 快速开始与基础
  - 安装与用法：https://tldraw.dev/installation
  - 基础组件示例（Tldraw）：https://tldraw.dev/examples/basic
  - 组件参考（Tldraw props）：https://tldraw.dev/reference/tldraw/Tldraw
  - TldrawUi 参考（UI Provider props）：https://tldraw.dev/reference/tldraw/TldrawUi
  - TldrawBaseProps（完整 props 列表）：https://tldraw.dev/reference/tldraw/TldrawBaseProps
  - 选项示例（options）：https://tldraw.dev/examples/custom-options
  - 本地持久化（persistenceKey 示例）：https://tldraw.dev/examples/persistence-key
  - Quick start（onMount / Editor 实例）：https://tldraw.dev/quick-start
  - Editor 概览与 useEditor：
    - https://tldraw.dev/docs/editor
    - Editor 参考：https://tldraw.dev/reference/editor/Editor
  - 富文本与 textOptions 示例（TipTap 扩展）：
    - https://tldraw.dev/examples/shapes/tools/rich-text-font-extensions

- 数据与持久化
  - 保存/加载快照（示例）：https://tldraw.dev/examples/snapshots
  - `loadSnapshot` 参考：https://tldraw.dev/reference/editor/loadSnapshot
  - Store API（`listen`、`mergeRemoteChanges` 等）：https://tldraw.dev/reference/store/Store
  - Store 事件示例：https://tldraw.dev/examples/store-events
  - Persistence（迁移指南）：https://tldraw.dev/docs/persistence
  - 自定义 shape 迁移（示例）：https://tldraw.dev/examples/shape-with-migrations
  - Persistence（persistenceKey / 迁移）：https://tldraw.dev/docs/persistence
  - Shape props 迁移示例：https://tldraw.dev/examples/shape-with-migrations
  - createShapePropsMigrationSequence：
    - https://tldraw.dev/reference/tlschema/createShapePropsMigrationSequence

- 协作能力
  - tldraw sync 文档：https://tldraw.dev/docs/sync
  - Cloudflare 自托管模板：https://github.com/tldraw/tldraw-sync-cloudflare
  - `TLSocketRoom` 参考：https://tldraw.dev/reference/sync-core/TLSocketRoom

- UI 与覆盖
  - User interface（hideUi / onUiEvent / 自定义 UI）：https://tldraw.dev/docs/user-interface
  - 向工具栏添加自定义工具（示例）：https://tldraw.dev/examples/add-tool-to-toolbar
  - TLUiOverrides 参考：
    - https://tldraw.dev/reference/tldraw/TLUiOverrides

- 相关生态（交叉）
  - MSW（接口模拟/流式）：`docs/msw/guides/streaming.md:1`
  - Playwright（Web-First 断言）：`docs/playwright/guides/web-first-assertions.md:1`
  - Vitest（Mock 与 Spy）：`docs/vitest/guides/mocking.md:1`

title: AI_GUIDE（AI 代理专用 · tldraw）
tool: tldraw
versions: [tldraw>=4, react>=18, node>=18]
status: stable
last_verified: 2025-10-02
audience: agent
tags: [agent, checklist, sync]

# AI_GUIDE（AI 代理专用 · tldraw）

## Must / Forbid
- Must：容器有显式尺寸；引入 `tldraw.css`。
- Must：通过 `onMount` 或 `useEditor()` 获取 `Editor`，写操作走 `editor.*` 或 `editor.store.*`。
- Must：持久化选型：`persistenceKey` 或 `getSnapshot/loadSnapshot`。
- Must：资产准备：默认 CDN；自托管时为 `<Tldraw assetUrls={...} />` 提供 `assetUrls`（参见安装文档三种方式：imports/imports.vite/urls/selfHosted）。
- Must（协作）：远端变更用 `editor.store.mergeRemoteChanges(() => { ... })` 应用，避免回环。
- Must（协作）：客户端与服务端注册一致的 `shapeUtils/bindingUtils/schema`；确保客户端与服务端 tldraw 版本匹配。
- Must（协作）：`useSync` 客户端需显式传入默认 + 自定义的 `shapeUtils/bindingUtils`（不会自动带入默认图形）。服务端用 `createTLSchema` + `defaultShapeSchemas/defaultBindingSchemas` + 自定义 shape 的 `props/migrations` 构建 schema，并传给 `TLSocketRoom`。
- Forbid：`useEditor()` 在 `<Tldraw />` 外部；直接改 DOM；在协作环境中裸调 `store.put/remove/update`（未包裹 `mergeRemoteChanges`）。
- Forbid：混用过时包（如旧的 `@tldraw/tldraw` 0.x）与新 API。

## 决策树（场景 → 推荐）
- 仅本地单人：
  - 最快：`<Tldraw persistenceKey="doc-key" />` 自动保持到 IndexedDB。
  - 可控：`getSnapshot/loadSnapshot` + 自己保存到 `localStorage` / 服务端。
- 原型协作：
  - `@tldraw/sync` demo（仅原型），或直接上 Cloudflare 模板（推荐）。
- 生产协作：
  - 部署 Cloudflare 模板（Durable Objects + R2），客户端用 `useSync` 创建 `store` 并传入 `<Tldraw store={store} />`。

## 执行步骤（最小闭环）
1) 安装并引入样式：`npm i tldraw`，`import 'tldraw/tldraw.css'`。
2) 资产：默认用 CDN；如自托管，用 `@tldraw/assets/imports(.vite)` / `@tldraw/assets/urls` / `@tldraw/assets/selfHosted` 生成 `assetUrls` 并传入。
3) 渲染：`<div style={{position:'fixed',inset:0}}><Tldraw assetUrls={assetUrls} /></div>`。
4) 获取 `editor`：在 `onMount` 或内部组件 `useEditor()`。
5) 本地持久化：
   - 自动：`persistenceKey="my-doc"`；或
   - 手动：`getSnapshot(editor.store)` → `loadSnapshot(editor.store, data)`。
6) 自定义扩展：注册 `shapeUtils` / `tools` / `overrides`（按需）。
7) 协作（可选）：部署 tldraw sync（Cloudflare 模板），客户端 `useSync({ uri, assets, shapeUtils })` → `<Tldraw store={store} />`。
8) 验收：刷新恢复、工具按钮可用、协作无回环（见 DoD）。

## DoD（验收标准）
- 画布可渲染，基础交互无报错；`useEditor()` 不越界使用。
- 刷新后数据可恢复（persistenceKey 或快照方案）。
- 自定义按钮能切换到自定义工具并创建目标图形。
- 协作：两端可见对方更改；远端变更通过 `mergeRemoteChanges` 应用；不出现重复触发监听。
- 链接均可点击直达官方文档；内部锚点可点击本仓库路径。

## 关键文件锚点
- 概览与最小示例：`docs/tldraw/README.md:1`
- 参考链接合集：`docs/tldraw/references.md:1`
- LLM 快速清单：`llms/tldraw-quickstart.llms.txt:1`

## 参考（URL）
- 安装与用法：https://tldraw.dev/installation
- 基础组件示例：https://tldraw.dev/examples/basic
- 快照保存/加载（示例 + API）：
  - https://tldraw.dev/examples/snapshots
  - https://tldraw.dev/reference/editor/loadSnapshot
- Store 参考（监听/合并远端更新）：https://tldraw.dev/reference/store/Store
- 协作（tldraw sync）：https://tldraw.dev/docs/sync
 - 选项示例（options）：https://tldraw.dev/examples/custom-options
 - User interface（hideUi / onUiEvent / 自定义 UI）：https://tldraw.dev/docs/user-interface
 - 向工具栏添加自定义工具：
   - https://tldraw.dev/examples/add-tool-to-toolbar
 - Editor 概览与 useEditor：
   - https://tldraw.dev/docs/editor

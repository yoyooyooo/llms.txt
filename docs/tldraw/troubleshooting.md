title: tldraw 故障排查
tool: tldraw
versions: [tldraw>=4, react>=18, node>=18]
status: stable
last_verified: 2025-10-02
tags: [troubleshooting]
audience: both

# tldraw 故障排查

## TL;DR
- 画布不显示/空白：父容器无显式尺寸或未引入 `tldraw.css`。
- 图标/字体 404：未传 `assetUrls` 或 bundler 未处理静态资源（.svg/.png/.json/.woff2）。
- `useEditor` 报错：Hook 只能在 `<Tldraw />` 内部组件使用。
- 刷新数据丢失：未配置 `persistenceKey` 或未实现快照保存/恢复。
- 协作回环/风暴：远端变更未用 `store.mergeRemoteChanges` 包裹。

## 常见问题 → 处理
- 容器尺寸问题
  - 现象：白屏或布局异常。
  - 处理：为父容器设置明确尺寸，例如 `style={{ position:'fixed', inset: 0 }}`。

- 样式未生效
  - 现象：UI 错乱、字体不对。
  - 处理：引入 `import 'tldraw/tldraw.css'`，必要时在全局 CSS 中 `@import 'tldraw/tldraw.css'`。

- 资产无法加载（404/跨域）
  - 现象：图标缺失、翻译不工作、控制台 404。
  - 处理：
    - 默认 CDN：无需配置；若公司网络限制，改为自托管。
    - bundler 导入：用 `@tldraw/assets/imports`（或 `imports.vite`）/ `urls` 生成 `assetUrls` 传给 `<Tldraw />`。
    - 自托管：复制官方仓库 assets 到 public，使用 `@tldraw/assets/selfHosted` 的 `getAssetUrls()`。
    - 打包器需处理 .svg/.png/.json/.woff2 静态资源。
    - 参考：`docs/tldraw/README.md:1` 和 https://tldraw.dev/installation

- `useEditor` 报错（上下文缺失）
  - 现象：Hook 抛错或返回空。
  - 处理：仅在 `<Tldraw />` 内部子组件中使用；或改用 `<Tldraw onMount={(editor)=>...} />`。

- 刷新后丢数据
  - 现象：刷新页面画布清空。
  - 处理：
    - 简易：`<Tldraw persistenceKey="my-doc" />`（IndexedDB）。
    - 手动：`getSnapshot(editor.store)` + `loadSnapshot(editor.store, data)`；示例：https://tldraw.dev/examples/snapshots

- 协作回环/监听风暴
  - 现象：来自远端的更新又被本地监听当作“用户变更”处理，导致循环。
  - 处理：将远端写入包裹在 `editor.store.mergeRemoteChanges(() => { /* put/remove/update */ })`。
  - 监听建议：`store.listen(handler, { source: 'user', scope: 'all' })`；示例：https://tldraw.dev/examples/store-events

- 去水印/许可
  - 现象：需要去除“Made with tldraw”。
  - 处理：购买商业授权，并在 `<Tldraw licenseKey="..." />` 传入；定价：https://tldraw.dev/#pricing

## 参考（URL）
- 安装与资产：
  - https://tldraw.dev/installation
  - https://tldraw.dev/docs/assets
- 持久化：
  - https://tldraw.dev/examples/snapshots
  - https://tldraw.dev/examples/persistence-key
- 事件与监听：
  - https://tldraw.dev/examples/store-events
- 协作：
  - https://tldraw.dev/docs/sync
  - https://github.com/tldraw/tldraw-sync-cloudflare

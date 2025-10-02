title: tldraw 实践总览
tool: tldraw
versions: [tldraw>=4, react>=18, node>=18]
status: stable
last_verified: 2025-10-02
tags: [whiteboard, canvas, editor, sync, custom-shape, custom-tool]
audience: both

# tldraw 实践总览

## TL;DR
- 安装 `tldraw` 并引入 `tldraw.css`，在有尺寸的容器内渲染 `<Tldraw />`。
- 通过 `onMount` 或 `useEditor` 获取 `Editor` 实例，所有编程式控制从此处开始。
- 持久化优先：`persistenceKey`（IndexedDB）或 `getSnapshot/loadSnapshot`（手动保存/恢复）。
- 深度扩展：自定义图形（`TLBaseShape` + `ShapeUtil`）与自定义工具（`StateNode`）。
- UI 定制：`components` / `overrides` / `hideUi` 三种方式组合使用。
- 协作：用 `@tldraw/sync`，生产环境建议部署 Cloudflare 模板后配置 `useSync`。
- 权威资料见“参考（URL）”；更多执行要点见 `docs/tldraw/AI_GUIDE.md:1`。

## 执行清单（Execution）
- 安装：`npm i tldraw`；引入样式：`import 'tldraw/tldraw.css'`。
- 在具有显式尺寸的容器内渲染 `<Tldraw />`。
- 在 `onMount` 或子组件的 `useEditor()` 中获取 `editor` 引用。
- 选择一种持久化方式：`persistenceKey` 或 `getSnapshot/loadSnapshot`。
- 如需自定义：注册 `shapeUtils` / `tools` / `overrides` 到 `<Tldraw />`。
- 如需协作：自托管 tldraw sync（Cloudflare 模板）并在客户端使用 `useSync`。
- 验收：刷新后数据可恢复；自定义工具可切换；协作无回环更新。

## 最小可运行示例（React）
```tsx
import { Tldraw, getSnapshot, loadSnapshot, useEditor } from 'tldraw'
import 'tldraw/tldraw.css'

export default function App() {
  return (
    <div style={{ position: 'fixed', inset: 0 }}>
      <Tldraw
        // 本地自动持久化（可选）：
        // persistenceKey="my-doc-key"
        components={{ SharePanel: SnapshotPanel }}
      />
    </div>
  )
}

function SnapshotPanel() {
  const editor = useEditor()

  return (
    <div style={{ padding: 12, pointerEvents: 'all', display: 'flex', gap: 8 }}>
      <button
        onClick={() => {
          const snapshot = getSnapshot(editor.store)
          localStorage.setItem('snapshot', JSON.stringify(snapshot))
        }}
      >保存快照</button>
      <button
        onClick={() => {
          const raw = localStorage.getItem('snapshot')
          if (!raw) return
          loadSnapshot(editor.store, JSON.parse(raw))
        }}
      >加载快照</button>
    </div>
  )
}
```

## 静态资产（Assets）与自托管
- 默认走官方 CDN，无需配置；要自托管或替换图标/字体/翻译时，向 `<Tldraw />` 传 `assetUrls`。

1) bundler 导入（webpack/rollup 等）
```ts
// A. 通用导入（需配置对 .svg/.png/.json/.woff2 的处理）
import { getAssetUrlsByImport } from '@tldraw/assets/imports'
const assetUrls = getAssetUrlsByImport()

// B. Vite（自动追加 ?url）
import { getAssetUrlsByImport } from '@tldraw/assets/imports.vite'
const assetUrls = getAssetUrlsByImport()

// C. 基于 import.meta.url 的标准写法
import { getAssetUrlsByMetaUrl } from '@tldraw/assets/urls'
const assetUrls = getAssetUrlsByMetaUrl()

// 传入组件
<Tldraw assetUrls={assetUrls} />
```

2) 完全自托管（复制仓库 assets 到你项目 public 路径）
```ts
import { getAssetUrls } from '@tldraw/assets/selfHosted'
const assetUrls = getAssetUrls() // 可传 { baseUrl } 指向你的 CDN 根路径
<Tldraw assetUrls={assetUrls} />
```

3) 定向覆盖单个资源
```ts
const assetUrls = { icons: { 'tool-hand': '/assets/custom-tool-hand.svg' } }
<Tldraw assetUrls={assetUrls} />
```

参考：
- 安装与资产说明：https://tldraw.dev/installation
- 资产/存储（`TLAssetStore`）：https://tldraw.dev/docs/assets

## 选项与许可（Options / License）
- `options` 控制编辑器行为（如限制页数、动画时长）。
```ts
import { Tldraw, TldrawOptions } from 'tldraw'
const options: Partial<TldrawOptions> = { maxPages: 3, animationMediumMs: 5000 }
<Tldraw options={options} />
```
- `licenseKey`：购买商业授权后传入可去除水印（详见定价页）。
  - 定价/许可：https://tldraw.dev/#pricing

## 事件与监听（Store Events）
```ts
// 仅监听用户来源、全量作用域
const cleanup = editor.store.listen((change) => {
  // 读取 added/updated/removed 记录，做外部同步
}, { source: 'user', scope: 'all' })
// 不用时调用 cleanup()
```
参考：存储事件示例：https://tldraw.dev/examples/store-events

## UI 定制（User Interface）
- 隐藏默认 UI：`<Tldraw hideUi />`（UI 与默认快捷键会被关闭，但可通过 `editor.setCurrentTool('draw')` 等 API 控制）。
- 覆盖 UI 行为：向 `<Tldraw />` 传 `overrides: TLUiOverrides`，如在 `tools()` 中新增自定义工具按钮、在 `toolbar()` 中插入位置。
- 覆盖 UI 组件：向 `<Tldraw />` 传 `components: TLComponents`，可替换默认 Toolbar/KeyboardShortcutsDialog 等。
- 监听 UI 事件：`onUiEvent={(name, data)=>{...}}` 仅在与 UI 交互时触发（手动调用 Editor API 不会触发）。

参考：
- User interface 文档（hideUi / onUiEvent / 自定义 UI）：https://tldraw.dev/docs/user-interface
- 向工具栏添加自定义工具（示例）：https://tldraw.dev/examples/add-tool-to-toolbar
- TLUiOverrides 参考：
  - https://tldraw.dev/reference/tldraw/TLUiOverrides

## 协作（useSync）关键要点
- 客户端：`useSync({ uri, assets, shapeUtils, bindingUtils })` 必须显式提供“默认 + 自定义”的 `shapeUtils/bindingUtils`；不会自动包含默认图形与绑定。
- 服务器：使用 `createTLSchema`（含 `defaultShapeSchemas/defaultBindingSchemas`）和自定义 shape 的 `props/migrations` 构建 schema，并传入 `TLSocketRoom`。确保每个房间仅一个 `TLSocketRoom` 实例。
- 书签形状（bookmark）：需在 `onMount` 时通过 `editor.registerExternalAssetHandler('url', unfurl)` 注册 unfurl 处理。
- 版本一致：客户端与服务端 tldraw 版本需匹配；升级时先更新后台再发新版前端。

参考：
- tldraw sync 指南：https://tldraw.dev/docs/sync
- `TLSocketRoom` 参考：https://tldraw.dev/reference/sync-core/TLSocketRoom

## Definition of Done（验收）
- 页面加载后能显示 tldraw 画布，基础绘制/选择操作无报错。
- 刷新后数据可通过 `persistenceKey` 或“保存/加载快照”按钮恢复。
- 可在 UI 上触发自定义功能（如按钮/工具）且行为正确。
- 若启用协作：两个浏览器会话能实时同步，无循环写入或异常。
- 文中链接可点击直达；可在 `docs/tldraw/AI_GUIDE.md:1` 找到机器执行清单。

## 参考（URL）
- 官方安装与用法：https://tldraw.dev/installation
- 基础组件示例（Tldraw）：https://tldraw.dev/examples/basic
- 保存/加载快照示例：
  - 文档与示例：https://tldraw.dev/examples/snapshots
  - `loadSnapshot` 参考：https://tldraw.dev/reference/editor/loadSnapshot
- Store 参考（监听、合并远端更新等）：https://tldraw.dev/reference/store/Store
- 协作（tldraw sync）：https://tldraw.dev/docs/sync
 - 选项示例（options）：https://tldraw.dev/examples/custom-options
 - 本地持久化（persistenceKey 示例）：https://tldraw.dev/examples/persistence-key
 - User interface（hideUi / onUiEvent / 自定义 UI）：https://tldraw.dev/docs/user-interface
 - 向工具栏添加自定义工具：
   - https://tldraw.dev/examples/add-tool-to-toolbar
 - Editor 概览与 useEditor：
   - https://tldraw.dev/docs/editor

## 相关
- 机器执行清单：`docs/tldraw/AI_GUIDE.md:1`
- 参考链接合集：`docs/tldraw/references.md:1`
- UI 覆盖指南：`docs/tldraw/guides/ui-overrides.md:1`
- 自定义 Shape：`docs/tldraw/guides/custom-shape.md:1`
- 自定义 Tool：`docs/tldraw/guides/custom-tool.md:1`
 - Shape 迁移：`docs/tldraw/guides/shape-props-migrations.md:1`
 - 拖拽绘制 Tool：`docs/tldraw/guides/tool-drag-to-create.md:1`
 - 形状属性迁移：`docs/tldraw/guides/shape-prop-migrations.md:1`
 - 拖拽绘制工具：`docs/tldraw/guides/drag-to-size-tool.md:1`
 - 协作端到端：`docs/tldraw/guides/collab-end-to-end.md:1`

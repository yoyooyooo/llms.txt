title: UI 覆盖与自定义按钮（tldraw）
tool: tldraw
versions: [tldraw>=4, react>=18]
status: stable
last_verified: 2025-10-02
tags: [ui, overrides, toolbar, components]
audience: both

# UI 覆盖与自定义按钮（tldraw）

## TL;DR
- 使用 `overrides: TLUiOverrides` 在 `tools()` 中注册自定义工具按钮；用 `components.Toolbar` 控制插入位置。
- 图标用 `assetUrls: TLUiAssetUrlOverrides` 提供；必要时配合 `hideUi` 与 `onUiEvent` 构建自定义 UI 流程。
- 传入 `<Tldraw tools={[...customTools]} overrides={uiOverrides} components={components} assetUrls={assetUrls} />`。

## 执行清单
- 定义工具（最小示例：点击画布创建一个 geo 图形）。
- 定义 `uiOverrides.tools` 新增工具按钮；定义 `components.Toolbar` 将按钮插入工具栏。
- 定义 `assetUrls.icons` 覆盖自定义图标。
- 在 `<Tldraw />` 中传入 `tools/overrides/components/assetUrls`。
- 验收：工具按钮显示、可切换、能创建图形。

## 最小示例
```tsx
import {
  StateNode,
  Tldraw,
  TLUiOverrides,
  TLComponents,
  TldrawUiMenuItem,
  useIsToolSelected,
  useTools,
} from 'tldraw'
import 'tldraw/tldraw.css'

// 1) 定义一个极简自定义工具：点击创建一个 geo 图形
class QuickGeoTool extends StateNode {
  static override id = 'quick-geo'
  override onPointerDown = () => {
    const p = this.editor.inputs.currentPagePoint
    this.editor.createShapes([
      { type: 'geo', x: p.x, y: p.y, props: { w: 120, h: 80 } },
    ])
  }
}

// 2) 注册 UI 覆盖：在 tools() 中挂载一个按钮定义
const uiOverrides: TLUiOverrides = {
  tools(editor, tools) {
    tools.quickGeo = {
      id: 'quick-geo',
      icon: 'quick-geo-icon',
      label: 'Quick Geo',
      kbd: 'g',
      onSelect: () => editor.setCurrentTool('quick-geo'),
    }
    return tools
  },
}

// 3) 覆盖 Toolbar 组件：插入自定义按钮
const components: TLComponents = {
  Toolbar: (props) => {
    const tools = useTools()
    const isSelected = useIsToolSelected(tools['quickGeo'])
    return (
      <div className="tlui-toolbar tlui-toolbar__top">
        <TldrawUiMenuItem {...tools['quickGeo']} isSelected={isSelected} />
        {props.children}
      </div>
    )
  },
}

// 4) 自定义图标（也可省略此步，直接用内置图标名如 'color'）
const assetUrls = {
  icons: { 'quick-geo-icon': '/icons/quick-geo.svg' },
}

// 5) 挂载到 Tldraw
export default function App() {
  return (
    <div style={{ position: 'fixed', inset: 0 }}>
      <Tldraw
        tools={[QuickGeoTool]}
        overrides={uiOverrides}
        components={components}
        assetUrls={assetUrls}
      />
    </div>
  )
}
```

## Definition of Done（验收）
- 自定义按钮出现在工具栏并支持快捷键；切换后能创建 geo 图形。
- 刷新后不报错；UI 自定义不影响基础交互。
- 参考链接可点击；代码可在 React 18 + tldraw>=4 下运行。

## 参考（URL）
- User interface（hideUi / onUiEvent / 自定义 UI）：https://tldraw.dev/docs/user-interface
- 向工具栏添加自定义工具（示例）：https://tldraw.dev/examples/add-tool-to-toolbar
- TLUiOverrides 参考：
  - https://tldraw.dev/reference/tldraw/TLUiOverrides

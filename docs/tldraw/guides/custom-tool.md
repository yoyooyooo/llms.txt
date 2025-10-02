title: 自定义 Tool（StateNode + 事件）
tool: tldraw
versions: [tldraw>=4, react>=18]
status: stable
last_verified: 2025-10-02
tags: [tool, statenode, events]
audience: both

# 自定义 Tool（StateNode + 事件）

## TL;DR
- 工具 = `StateNode` 子类；在 `onPointerDown/Move/Up` 等事件里调用 `editor.*` 完成交互。
- 在 `<Tldraw tools={[MyTool]} />` 注册；用 `overrides.tools/toolbar` 或自定义 `components.Toolbar` 暴露按钮。
- 可与自定义 Shape 协同：工具负责创建/更新该 Shape；UI 负责选择/切换工具。

## 执行清单
- 定义一个继承 `StateNode` 的工具类（设置 `static id`）。
- 实现最小事件（如 `onPointerDown`）并在其中调用 `editor.createShapes/updateShapes`。
- 在 `<Tldraw tools={[MyTool]} />` 注册；使用 `overrides` 或 `components.Toolbar` 添加按钮。
- 验证指针交互、撤销/重做、键鼠配合是否符合预期。

## 最小示例（点击落点创建自定义 Note）
```tsx
import { StateNode, Tldraw, TLUiOverrides, TLComponents, TldrawUiMenuItem, useTools, useIsToolSelected } from 'tldraw'
import 'tldraw/tldraw.css'

// 1) 工具：在点击处创建一个 geo 矩形（可对接自定义 Shape）
class QuickBoxTool extends StateNode {
  static override id = 'quick-box'
  override onPointerDown = () => {
    const p = this.editor.inputs.currentPagePoint
    this.editor.createShapes([{ type: 'geo', x: p.x, y: p.y, props: { w: 160, h: 100 } }])
  }
}

// 2) UI 覆盖：注册一个工具按钮（见更完整示例：ui-overrides 指南）
const uiOverrides: TLUiOverrides = {
  tools(editor, tools) {
    tools.quickBox = {
      id: 'quick-box',
      icon: 'color',
      label: 'Quick Box',
      kbd: 'b',
      onSelect: () => editor.setCurrentTool('quick-box'),
    }
    return tools
  },
}

const components: TLComponents = {
  Toolbar: (props) => {
    const tools = useTools()
    const isSelected = useIsToolSelected(tools['quickBox'])
    return (
      <div className="tlui-toolbar tlui-toolbar__top">
        <TldrawUiMenuItem {...tools['quickBox']} isSelected={isSelected} />
        {props.children}
      </div>
    )
  },
}

export default function App() {
  return (
    <div style={{ position: 'fixed', inset: 0 }}>
      <Tldraw tools={[QuickBoxTool]} overrides={uiOverrides} components={components} />
    </div>
  )
}
```

## 与自定义 Shape 协同
- 工具只负责交互/创建/更新；具体渲染/几何/缩放交给 `ShapeUtil`（参考：`custom-shape.md`）。
- 当工具服务于自定义 shape 时：
  - 创建时 `editor.createShapes([{ type: 'note', ... }])`；
  - 更新时 `editor.updateShapes([{ id, type: 'note', props: { ... } }])`。

## Definition of Done（验收）
- 工具按钮可选中并可用快捷键切换；点击画布可创建图形；撤销/重做有效。
- 与 UI 覆盖/自定义 Shape 组合使用时，行为一致无冲突。

## 参考（URL）
- 自定义工具（贴纸示例入口）：https://tldraw.dev/examples/custom-tool
- BaseBoxShapeTool（工具基类参考）：https://tldraw.dev/reference/editor/BaseBoxShapeTool
- UI 覆盖与按钮：`docs/tldraw/guides/ui-overrides.md:1`
- 自定义 Shape：`docs/tldraw/guides/custom-shape.md:1`

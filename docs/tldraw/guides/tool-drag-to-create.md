title: 拖拽绘制矩形（自定义 Tool）
tool: tldraw
versions: [tldraw>=4, react>=18]
status: stable
last_verified: 2025-10-02
tags: [tool, drag, createShapeId]
audience: both

# 拖拽绘制矩形（自定义 Tool）

## TL;DR
- 工具在 `onPointerDown` 记录起点并创建一个 1×1 矩形；`onPointerMove` 动态更新 `x/y/w/h`；`onPointerUp` 完成并选中。
- 使用 `createShapeId()` 生成确定的 `id`，方便后续 `updateShapes`。
- 与 UI overrides 结合，添加按钮便于切换。

## 执行清单
- 定义 `DragRectTool extends StateNode`，维护 `start` 与 `shapeId`。
- `onPointerDown`：记起点 + 创建 shape；`onPointerMove`：更新位置尺寸；`onPointerUp`：结束并选中。
- 在 `<Tldraw tools={[DragRectTool]} overrides={...} components={...} />` 注册工具与按钮。

## 最小示例
```tsx
import { StateNode, Tldraw, TLUiOverrides, TLComponents, TldrawUiMenuItem, useTools, useIsToolSelected, createShapeId } from 'tldraw'
import 'tldraw/tldraw.css'

class DragRectTool extends StateNode {
  static override id = 'drag-rect'
  private start?: { x: number; y: number }
  private shapeId?: string

  override onPointerDown = () => {
    const p = this.editor.inputs.currentPagePoint
    this.start = { x: p.x, y: p.y }
    const id = createShapeId()
    this.shapeId = id
    this.editor.createShapes([{ id, type: 'geo', x: p.x, y: p.y, props: { w: 1, h: 1 } }])
  }

  override onPointerMove = () => {
    if (!this.start || !this.shapeId) return
    const p = this.editor.inputs.currentPagePoint
    const minX = Math.min(this.start.x, p.x)
    const minY = Math.min(this.start.y, p.y)
    const w = Math.max(1, Math.abs(p.x - this.start.x))
    const h = Math.max(1, Math.abs(p.y - this.start.y))
    this.editor.updateShapes([
      { id: this.shapeId as any, type: 'geo', x: minX, y: minY, props: { w, h } },
    ])
  }

  override onPointerUp = () => {
    if (this.shapeId) {
      this.editor.select(this.shapeId as any)
      this.editor.markHistoryStoppingPoint()
    }
    this.start = undefined
    this.shapeId = undefined
  }
}

const uiOverrides: TLUiOverrides = {
  tools(editor, tools) {
    tools.dragRect = {
      id: 'drag-rect',
      icon: 'color',
      label: 'Drag Rect',
      kbd: 'r',
      onSelect: () => editor.setCurrentTool('drag-rect'),
    }
    return tools
  },
}

const components: TLComponents = {
  Toolbar: (props) => {
    const tools = useTools()
    const isSelected = useIsToolSelected(tools['dragRect'])
    return (
      <div className="tlui-toolbar tlui-toolbar__top">
        <TldrawUiMenuItem {...tools['dragRect']} isSelected={isSelected} />
        {props.children}
      </div>
    )
  },
}

export default function App() {
  return (
    <div style={{ position: 'fixed', inset: 0 }}>
      <Tldraw tools={[DragRectTool]} overrides={uiOverrides} components={components} />
    </div>
  )
}
```

## Definition of Done（验收）
- 工具按钮可用；拖拽可创建矩形并在拖动中实时改变尺寸；松开后选中目标并形成历史断点。
- 撤销/重做有效；与 UI 覆盖/自定义 Shape 组合时行为一致。

## 参考（URL）
- Editor 概览（createShapes/updateShapes）：https://tldraw.dev/docs/editor
- UI 覆盖与按钮：`docs/tldraw/guides/ui-overrides.md:1`

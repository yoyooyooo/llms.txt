title: 拖拽绘制工具（从按下到拖动到释放）
tool: tldraw
versions: [tldraw>=4, react>=18]
status: stable
last_verified: 2025-10-02
tags: [tool, draw, pointer]
audience: both

# 拖拽绘制工具（从按下到拖动到释放）

## TL;DR
- 基于 `StateNode` 实现“按下创建初始图形→移动更新尺寸→释放完成”的绘制流。
- 使用 `editor.inputs.currentPagePoint` 作为参考；用 `updateShapes` 实时更新 `props.w/h`。
- 配合 UI 覆盖添加工具按钮与快捷键。

## 执行清单
- 定义工具类（`static id` 唯一）。
- `onPointerDown`：记录起点，创建初始形状。
- `onPointerMove`：计算差值更新 `w/h`（注意最小值与反向拖动）。
- `onPointerUp`：结束；必要时归一化坐标与尺寸。
- 注册到 `<Tldraw tools={[...]}/>` 并在 UI 中暴露按钮。

## 最小示例
```tsx
import { StateNode, Tldraw, TLUiOverrides, TLComponents, TldrawUiMenuItem, useTools, useIsToolSelected } from 'tldraw'
import 'tldraw/tldraw.css'

class DragRectTool extends StateNode {
  static override id = 'drag-rect'

  private start = { x: 0, y: 0 }
  private shapeId: string | null = null

  override onPointerDown = () => {
    const p = this.editor.inputs.currentPagePoint
    this.start = { x: p.x, y: p.y }
    const shape = this.editor.createShape({ type: 'geo', x: p.x, y: p.y, props: { w: 1, h: 1 } })
    this.shapeId = shape[0].id
  }

  override onPointerMove = () => {
    if (!this.shapeId) return
    const p = this.editor.inputs.currentPagePoint
    const shift = this.editor.inputs.shiftKey // Shift 固定比例
    let x = this.start.x
    let y = this.start.y
    let w = p.x - this.start.x
    let h = p.y - this.start.y
    // 处理反向拖动
    if (w < 0) { x += w; w = Math.abs(w) }
    if (h < 0) { y += h; h = Math.abs(h) }
    if (shift) { const s = Math.max(w, h); w = s; h = s }
    this.editor.updateShapes([{ id: this.shapeId, type: 'geo', x, y, props: { w: Math.max(1, w), h: Math.max(1, h) } }])
  }

  override onPointerUp = () => {
    this.shapeId = null
  }

  // Esc 取消：删除正在创建的图形
  override onKeyDown = (info: any) => {
    if (info.key === 'Escape' && this.shapeId) {
      this.editor.deleteShapes([this.shapeId])
      this.shapeId = null
      return
    }
  }
}

const overrides: TLUiOverrides = {
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
      <Tldraw tools={[DragRectTool]} overrides={overrides} components={components} />
    </div>
  )
}
```

## 注意事项
- 反向拖动时需要重置 `x/y` 以保持 `w/h` 为正值；也可通过变换矩阵实现更高级的效果。
- 若与自定义 Shape 结合，将 `type: 'geo'` 替换为你的自定义 shape type，并在 `updateShapes` 中更新自定义 props。
- 为了更顺滑的体验，可用 `editor.history.batch` 合并中间过程为单次撤销，或在按下时 `markHistoryStoppingPoint()`；Esc 取消当前绘制，Shift 固定比例。

## Definition of Done（验收）
- 工具按钮可切换；按下-拖动-释放流工作正常，支持反向拖动与最小尺寸。
- 撤销/重做行为符合预期（拖拽整个过程合并为一组）。

## 参考（URL）
- BaseBoxShapeTool（工具基类参考）：https://tldraw.dev/reference/editor/BaseBoxShapeTool
- UI 覆盖与按钮：`docs/tldraw/guides/ui-overrides.md:1`

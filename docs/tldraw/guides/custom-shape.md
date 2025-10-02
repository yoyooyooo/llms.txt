title: 自定义 Shape（ShapeUtil + 迁移）
tool: tldraw
versions: [tldraw>=4, react>=18]
status: stable
last_verified: 2025-10-02
tags: [shape, shapeutil, migrations]
audience: both

# 自定义 Shape（ShapeUtil + 迁移）

## TL;DR
- 定义类型：`TLBaseShape<'my-shape', Props>`；实现 `ShapeUtil` 或继承 `BaseBoxShapeUtil` 简化几何与缩放。
- 在 util 中提供 `static type/static props/getDefaultProps/component/indicator`；必要时实现 `getGeometry/onResize`。
- 在 `<Tldraw shapeUtils={[MyShapeUtil]} />` 注册；UI 按钮见 `docs/tldraw/guides/ui-overrides.md:1`。
- 协作：服务端用 `createTLSchema` 合并 `defaultShapeSchemas` + 自定义 `props/migrations`；客户端/服务端保持一致。

## 执行清单
- 定义 `TLBaseShape` 类型与 `props`（用 `T` / `RecordProps` 校验）。
- 选择继承 `ShapeUtil` 或 `BaseBoxShapeUtil`；实现渲染/指示器/几何/缩放等。
- 在 `<Tldraw />` 注册 `shapeUtils` 并验证创建/选择/缩放。
- 协作场景：为服务端 schema 提供自定义 `props/migrations`，客户端 `useSync` 也传自定义 `shapeUtils`。

## 最小示例（BaseBoxShapeUtil 简化版）
```tsx
import {
  TLBaseShape,
  BaseBoxShapeUtil,
  HTMLContainer,
  Rectangle2d,
  T,
  RecordProps,
  Tldraw,
} from 'tldraw'
import 'tldraw/tldraw.css'

// 1) 类型定义
type NoteShape = TLBaseShape<'note', { w: number; h: number; text: string }>

// 2) ShapeUtil（基于 BaseBoxShapeUtil 省去 getGeometry/onResize）
class NoteShapeUtil extends BaseBoxShapeUtil<NoteShape> {
  static override type = 'note' as const
  static override props: RecordProps<NoteShape> = {
    w: T.number,
    h: T.number,
    text: T.string,
  }
  override getDefaultProps(): NoteShape['props'] {
    return { w: 200, h: 120, text: 'Hello Note' }
  }
  override getGeometry(shape: NoteShape) {
    return new Rectangle2d({ width: shape.props.w, height: shape.props.h, isFilled: true })
  }
  override component(shape: NoteShape) {
    return (
      <HTMLContainer
        style={{
          background: '#FFFAAD',
          border: '1px solid #E0D96C',
          padding: 8,
          width: shape.props.w,
          height: shape.props.h,
        }}
      >
        {shape.props.text}
      </HTMLContainer>
    )
  }
  override indicator(shape: NoteShape) {
    return <rect width={shape.props.w} height={shape.props.h} rx={4} ry={4} />
  }
}

// 3) 注册并创建一个 shape
export default function App() {
  return (
    <div style={{ position: 'fixed', inset: 0 }}>
      <Tldraw
        shapeUtils={[NoteShapeUtil]}
        onMount={(editor) => {
          editor.createShape({ type: 'note', x: 160, y: 160 })
        }}
      />
    </div>
  )
}
```

## 协作要点（服务端 schema）
- 服务端构建 schema：
  - `createTLSchema({ shapes: { ...defaultShapeSchemas, note: { props: /* 与上面一致 */, migrations: /* 可选 */ } } })`
  - 传入 `TLSocketRoom({ schema })`；客户端 `useSync({ shapeUtils: [...defaultShapeUtils, NoteShapeUtil] })`。
- 如省略 `props` 将缺少服务端校验；如省略 `migrations` 不同版本协作时可能失败。

## Definition of Done（验收）
- 画布可创建/选择/缩放自定义 `note`；指示器与尺寸变化正确。
- 刷新后不报错；与 UI 覆盖结合时能从工具栏激活创建（见 ui-overrides 指南）。
- 协作：两端能同步自定义 shape；服务端 schema 校验通过，无迁移错误。

## 参考（URL）
- 自定义 shape 示例：https://tldraw.dev/examples/custom-shape
- BaseBoxShapeUtil 参考：https://tldraw.dev/reference/editor/BaseBoxShapeUtil
- BaseBoxShapeTool（工具基类参考）：https://tldraw.dev/reference/editor/BaseBoxShapeTool
- 协作与 schema：https://tldraw.dev/docs/sync
- TLSocketRoom 参考：https://tldraw.dev/reference/sync-core/TLSocketRoom
- UI 覆盖与按钮：`docs/tldraw/guides/ui-overrides.md:1`

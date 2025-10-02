title: Shape Props 迁移（版本兼容）
tool: tldraw
versions: [tldraw>=4, react>=18]
status: stable
last_verified: 2025-10-02
tags: [migrations, persistence, schema]
audience: both

# Shape Props 迁移（版本兼容）

## TL;DR
- 为自定义 Shape 定义迁移序列：`createShapePropsMigrationIds` + `createShapePropsMigrationSequence`。
- 在 `ShapeUtil` 上声明 `static migrations = ...`；旧快照载入时自动迁移。
- 协作/服务端：`createTLSchema({ migrations: [myMigrations] })`；客户端 `createTLStore` 或 `<Tldraw migrations={[myMigrations]} />`。

## 执行清单
- 定义迁移 ID（与 Shape 类型一致，如 `'note'`）。
- 定义迁移序列：每个迁移包含 `id`、`up(props)`、`down(props)`。
- 在自定义 `ShapeUtil` 上设置 `static migrations`。
- 载入旧快照验证自动升级；必要时在 store/schema 层追加 `migrations`。

## 最小示例（新增 color 字段）
```tsx
import {
  BaseBoxShapeUtil,
  HTMLContainer,
  T,
  TLBaseShape,
  TLResizeInfo,
  TLStoreSnapshot,
  Tldraw,
  createShapePropsMigrationIds,
  createShapePropsMigrationSequence,
  resizeBox,
} from 'tldraw'
import 'tldraw/tldraw.css'

type NoteShape = TLBaseShape<'note', { w: number; h: number; color: string }>

// 1) 迁移 ID（与形状类型匹配）
const versions = createShapePropsMigrationIds('note', { AddColor: 1 })

// 2) 迁移序列（为 props 添加 color）
export const noteShapeMigrations = createShapePropsMigrationSequence({
  sequence: [
    {
      id: versions.AddColor,
      up(props) {
        props.color = 'lightyellow'
      },
      down(props) {
        delete (props as any).color
      },
    },
  ],
})

export class NoteShapeUtil extends BaseBoxShapeUtil<NoteShape> {
  static override type = 'note' as const
  static override props = { w: T.number, h: T.number, color: T.string }
  // 3) 声明迁移
  static override migrations = noteShapeMigrations

  override getDefaultProps(): NoteShape['props'] {
    return { w: 240, h: 140, color: 'lightyellow' }
  }
  override component(shape: NoteShape) {
    return (
      <HTMLContainer id={shape.id} style={{ backgroundColor: shape.props.color }} />
    )
  }
  override indicator(shape: NoteShape) {
    return <rect width={shape.props.w} height={shape.props.h} />
  }
  override onResize(shape: NoteShape, info: TLResizeInfo<NoteShape>) {
    return resizeBox(shape, info)
  }
}

const utils = [NoteShapeUtil]

export default function Example({ snapshot }: { snapshot?: TLStoreSnapshot }) {
  return (
    <div className="tldraw__editor">
      <Tldraw shapeUtils={utils} snapshot={snapshot} />
    </div>
  )
}
```

## 在 Store / Schema 注入迁移（可选）
- `<Tldraw migrations={[noteShapeMigrations]} />`
- `createTLStore({ migrations: [noteShapeMigrations] })`
- `createTLSchema({ migrations: [noteShapeMigrations] })`

## Definition of Done（验收）
- 加载旧版快照（不含 color）后自动迁移到含 color；颜色渲染正确。
- 回滚（down）逻辑能使新版本快照迁移为旧版本（在需要时）。
- 协作：客户端与服务端均注册相同迁移，版本不匹配时不报错；两端数据一致。

## 参考（URL）
- Persistence（迁移指南）：https://tldraw.dev/docs/persistence
- 自定义 shape 迁移示例：
  - https://tldraw.dev/examples/shape-with-migrations
- BaseBoxShapeUtil 参考：https://tldraw.dev/reference/editor/BaseBoxShapeUtil

title: 形状属性迁移（Shape Props Migrations）
tool: tldraw
versions: [tldraw>=4]
status: stable
last_verified: 2025-10-02
tags: [migrations, persistence, schema]
audience: both

# 形状属性迁移（Shape Props Migrations）

## TL;DR
- 为自定义 Shape 的 `props` 演进提供向上/向下迁移，避免旧快照或协作中版本不一致导致崩溃。
- 在自定义 `ShapeUtil` 上定义 `static migrations`，使用 `createShapePropsMigrationIds` 与 `createShapePropsMigrationSequence`。
- 迁移函数仅接收 `props`，可原地修改（不需要返回新对象）。
- 服务端 schema 与客户端 `shapeUtils` 的迁移定义需一致；快照加载时会自动执行迁移。

## 执行清单
- 为每个 shape 定义唯一的迁移序列 ID（需与 shape type 一致），版本号从 1 开始、按序递增。
- 使用 `createShapePropsMigrationSequence` 编写 `up` / （可选）`down` 逻辑。
- 在自定义 `ShapeUtil` 上挂载 `static migrations = ...`。
- 协作/服务端：用 `createTLSchema` 构建 schema，包含该 shape 的 `props` 与 `migrations`。
- 验收：加载旧快照能自动迁移；协作端到端不报错。

## 最小示例
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

// 1) 定义 Shape 类型（新增 color 字段）
type Card = TLBaseShape<'card', { w: number; h: number; color: string }>

// 2) 声明迁移 ID（与 shape type 匹配；版本从 1 开始）
const Versions = createShapePropsMigrationIds('card', {
  AddColor: 1,
})

// 3) 定义迁移序列（props-only；可原地修改）
const cardMigrations = createShapePropsMigrationSequence({
  sequence: [
    {
      id: Versions.AddColor,
      up(props) {
        if (props.color == null) props.color = 'lightblue'
      },
      down(props) {
        delete (props as any).color
      },
    },
  ],
})

// 4) 自定义 ShapeUtil，挂载 migrations
class CardShapeUtil extends BaseBoxShapeUtil<Card> {
  static override type = 'card' as const
  static override props = { w: T.number, h: T.number, color: T.string }
  static override migrations = cardMigrations

  override getDefaultProps(): Card['props'] {
    return { w: 240, h: 140, color: 'lightblue' }
  }
  override onResize(shape: Card, info: TLResizeInfo<Card>) { return resizeBox(shape, info) }
  override component(shape: Card) {
    return (
      <HTMLContainer style={{ width: shape.props.w, height: shape.props.h, background: shape.props.color }} />
    )
  }
  override indicator(shape: Card) { return <rect width={shape.props.w} height={shape.props.h} /> }
}

// 5) 旧快照在加载时将自动执行迁移
export default function App({ snapshot }: { snapshot?: TLStoreSnapshot }) {
  return (
    <div style={{ position: 'fixed', inset: 0 }}>
      <Tldraw shapeUtils={[CardShapeUtil]} snapshot={snapshot} />
    </div>
  )
}
```

## 字段重命名 / 类型变更示例
```ts
import { createShapePropsMigrationIds, createShapePropsMigrationSequence } from 'tldraw'

// 场景 A：将旧字段 bg 重命名为 color（可回退）
const V1 = createShapePropsMigrationIds('card', { RenameBgToColor: 1 })
export const cardMigrations = createShapePropsMigrationSequence({
  sequence: [
    {
      id: V1.RenameBgToColor,
      up(props) {
        if ((props as any).bg && props.color == null) props.color = (props as any).bg
        delete (props as any).bg
      },
      down(props) {
        if ((props as any).color && (props as any).bg == null) (props as any).bg = (props as any).color
        delete (props as any).color
      },
    },
  ],
})

// 场景 B：将 size:number 拆为 w/h（不可回退示例）
const V2 = createShapePropsMigrationIds('box', { SplitSizeToWH: 1 })
export const boxMigrations = createShapePropsMigrationSequence({
  sequence: [
    {
      id: V2.SplitSizeToWH,
      up(props) {
        const size = (props as any).size
        if (typeof size === 'number') {
          ;(props as any).w = size
          ;(props as any).h = size
          delete (props as any).size
        }
      },
      // 如需回退，可在 down 中将 w/h 合并为 size
    },
  ],
})
```

## 离线迁移与版本采集
- 离线迁移而不加载到 Editor：
```ts
import { createTLSchema } from 'tldraw'

const result = createTLSchema({ /* shapes/bindings/migrations */ }).migrateStoreSnapshot(snapshot)
if (result.type === 'success') {
  const migrated = result.value
  // 保存 migrated 以供后续加载
}
```
- 采集当前 schema 版本（建议与快照一起持久化一次即可）：
```ts
const schemaVersion = editor.store.schema.serialize()
// 将 schemaVersion 存入快照，便于下次加载时对比并执行迁移
```

## 服务端 schema 与协作
- 使用 `createTLSchema({ shapes: { card: { props: /* 与上文一致 */, migrations: cardMigrations } } })`；
- 通过 `TLSocketRoom({ schema })` 应用；客户端 `useSync` 需传入 `shapeUtils: [...defaultShapeUtils, CardShapeUtil]`。
- 版本管理：新增迁移一旦发布不可重排/重命名；不同客户端版本通过 up/down 兼容。

## Definition of Done（验收）
- 旧快照加载后能自动补全/迁移 `props`，不报错；移除迁移后回退成功（如实现了 down）。
- 协作场景：两端/服务端 schema 一致，版本不匹配时仍能通过迁移正常协作。

## 参考（URL）
- Persistence（持久化/迁移）：https://tldraw.dev/docs/persistence
- 自定义 shape 迁移示例：https://tldraw.dev/examples/shape-with-migrations
- createShapePropsMigrationSequence：
  - https://tldraw.dev/reference/tlschema/createShapePropsMigrationSequence
- createTLSchema（聚合 schema/migrations）：
  - https://tldraw.dev/reference/tlschema/createTLSchema

## 进阶：字段重命名/类型变更与“离线迁移”

### 字段重命名/类型变更（up/down 双向）
```ts
// 假设 v1: props.title: string；v2: props.text: string，且 color 从 number → string
const Versions = createShapePropsMigrationIds('card', { RenameTitleToText: 1, ColorNumToStr: 2 })

const migrations = createShapePropsMigrationSequence({
  sequence: [
    {
      id: Versions.RenameTitleToText,
      up(props) {
        if ((props as any).title != null && (props as any).text == null) {
          ;(props as any).text = (props as any).title
          delete (props as any).title
        }
      },
      down(props) {
        if ((props as any).text != null && (props as any).title == null) {
          ;(props as any).title = (props as any).text
          delete (props as any).text
        }
      },
    },
    {
      id: Versions.ColorNumToStr,
      up(props) {
        if (typeof (props as any).color === 'number') {
          ;(props as any).color = String((props as any).color)
        }
      },
      down(props) {
        if (typeof (props as any).color === 'string' && /^\d+$/.test((props as any).color)) {
          ;(props as any).color = Number((props as any).color)
        }
      },
    },
  ],
})
```

### 离线迁移（不挂载到编辑器时对快照执行迁移）
```ts
import { createTLSchema } from 'tldraw'

async function migrateSnapshotOffline(snapshot: any) {
  const schema = createTLSchema({ /* 按需传入 migrations/sequences */ })
  const res = schema.migrateStoreSnapshot(snapshot)
  if (res.type === 'success') return res.value
  throw new Error(`Migration failed: ${res.reason}`)
}
```

提示：若你的服务端需要批量升级历史快照，可将以上离线迁移作为一条数据任务执行。

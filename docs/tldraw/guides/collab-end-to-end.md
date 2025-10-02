title: 协作端到端（Cloudflare + useSync + 资产存储）
tool: tldraw
versions: [tldraw>=4]
status: stable
last_verified: 2025-10-02
tags: [sync, collab, schema, assets]
audience: both

# 协作端到端（Cloudflare + useSync + 资产存储）

## TL;DR
- 后端：基于官方 Cloudflare 模板部署 `tldraw sync`（Durable Objects + R2）；确保单房间单 `TLSocketRoom`。
- 客户端：`useSync({ uri, assets, shapeUtils, bindingUtils })`；显式传“默认 + 自定义”的 shape/binding utils。
- Schema：服务端用 `createTLSchema` 合并默认与自定义 schema（含 `props/migrations`），客户端版本与服务端匹配。
- 资产：实现 `TLAssetStore` 上传/解析（bookmark 需 `registerExternalAssetHandler('url', ...)`）。

## 执行清单
- 部署：克隆并部署 Cloudflare 模板（Workers + DO + R2）。
- 服务端：在 Worker 端创建 `TLSocketRoom`，传入 `schema`；定期快照存 R2；保证每个房间仅一个 DO 实例。
- 客户端：在页面中调用 `useSync({ uri, assets, shapeUtils, bindingUtils })` 获取 store；把 store 传给 `<Tldraw store={store} />`。
- 自定义数据：客户端与服务端同时注册自定义 shape/binding 的 util/schema；必要时实现 props 迁移。
- 验收：两端实时同步；刷新/重连后恢复；无回环更新；大文件通过资产存储走上传。

## 客户端最小示例（useSync + 资产存储）
```tsx
import { Tldraw, TLAssetStore, defaultShapeUtils, defaultBindingUtils, Editor } from 'tldraw'
import { useSync } from '@tldraw/sync'
import 'tldraw/tldraw.css'
import { MyShapeUtil, MyBindingUtil } from './custom'

const myAssetStore: TLAssetStore = {
  async upload(file, asset) {
    const url = await uploadToMyWorker(file) // 返回可访问 URL
    return url
  },
  async resolve(asset) {
    return asset.props.src // or fetch signed URL
  },
}

export default function App({ roomId }: { roomId: string }) {
  const store = useSync({
    uri: `wss://YOUR-WORKER-DOMAIN/connect/${roomId}`,
    assets: myAssetStore,
    shapeUtils: [...defaultShapeUtils, MyShapeUtil],
    bindingUtils: [...defaultBindingUtils, MyBindingUtil],
  })

  function registerBookmark(editor: Editor) {
    editor.registerExternalAssetHandler('url', async ({ url }) => await unfurl(url))
  }

  return <Tldraw store={store} onMount={registerBookmark} />
}
```

## 服务端（schema + TLSocketRoom）
- 用 `createTLSchema({ shapes, bindings, migrations })` 聚合默认 + 自定义 schema 信息；传给 `TLSocketRoom({ schema })`。
- 参考模板：
  - https://github.com/tldraw/tldraw-sync-cloudflare
  - `TLSocketRoom` 参考：https://tldraw.dev/reference/sync-core/TLSocketRoom

### Cloudflare Worker 端（简化片段）
```ts
// worker/TldrawDurableObject.ts（示意）
import { TLSocketRoom } from '@tldraw/sync-core'
import { createTLSchema, defaultShapeSchemas, defaultBindingSchemas } from '@tldraw/tlschema'

export class TldrawDurableObject {
  room: TLSocketRoom<any, any>

  constructor(state: DurableObjectState, env: Env) {
    const schema = createTLSchema({
      shapes: { ...defaultShapeSchemas /* , card: { props, migrations } */ },
      bindings: { ...defaultBindingSchemas },
      // migrations: [/* 如需 meta 级迁移 */],
    })
    this.room = new TLSocketRoom({ schema /* , initialSnapshot */ })
  }

  async fetch(req: Request) {
    const upgradeHeader = req.headers.get('Upgrade')
    if (upgradeHeader === 'websocket') {
      const [client, server] = Object.values(new WebSocketPair()) as [WebSocket, WebSocket]
      await this.room.handleSocketConnect(server as any, { roomId: 'id-from-url' })
      return new Response(null, { status: 101, webSocket: client })
    }
    return new Response('OK')
  }
}
```

提示：实际模板还包含 R2 持久化、消息转发（handleSocketMessage/Close/Error）、以及定期保存快照等逻辑。

## Definition of Done（验收）
- 两端可实时看到彼此操作；断线重连后文档一致。
- 资产上传/解析可用；bookmark 正确生成。
- 协作中自定义 shape/binding 正常工作；迁移可跨版本协作无报错。

## 参考（URL）
- tldraw sync 指南：https://tldraw.dev/docs/sync
- Cloudflare 模板：https://github.com/tldraw/tldraw-sync-cloudflare
- `TLSocketRoom` 参考：https://tldraw.dev/reference/sync-core/TLSocketRoom
 - Cloudflare Durable Objects 入门：https://developers.cloudflare.com/durable-objects/get-started/

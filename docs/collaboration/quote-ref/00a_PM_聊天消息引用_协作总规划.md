# Lumen / 陆小七 · 聊天消息引用功能协作规划 v0.1

> 面向对象：后端模型层 + 安卓前端  
> 功能主题：聊天消息引用 Quote / Reply-to  
> PM 结论：本功能应作为当前 Chat MVP 的下一项基础能力推进。先实现“引用聊天消息 / 图片消息”，为后续引用朋友圈、日记、相册图片预留结构。

---

## 1. 为什么要做消息引用

消息引用不是一个普通聊天小功能。对 Lumen 来说，它是后续“朋友圈 / 日记 / 照片反咬聊天”的基础数据结构。

它将支持：

```text
1. 用户回复某条旧消息，小七能知道用户在说哪一句。
2. 用户回复某张图，小七能知道图的来源和 caption。
3. 未来用户回复某条朋友圈，小七能接上当时的生活状态。
4. 未来用户引用日记句子，小七能根据权限决定说不说。
5. 后端可以把引用内容作为强上下文注入，减少误解和幻觉。
```

当前先做 P0：

```text
引用 message，包括纯文字 message、图文 message、纯图 message。
```

未来预留：

```text
moment
diary
image
```

---

## 2. 当前系统现状

当前后端消息协议主字段是：

```json
{
  "id": 1,
  "role": "user | assistant",
  "content": "...",
  "image_url": "/uploads/...",
  "created_at": "iso8601"
}
```

当前 SSE 已支持：

```text
delta
bubble_end
tool_call
tool_result
image
error
done
```

当前图片消息已经是 `content + image_url` 单气泡双载荷，caption 可能为空。

引用功能必须兼容这些现状，不要推翻现有 Chat / Image 链路。

---

## 3. 功能范围

### P0 必做

```text
1. 用户长按任意聊天消息，出现“引用”操作。
2. Composer 上方显示引用预览条。
3. 用户发送新消息时携带 quote_ref。
4. 后端保存 quote_ref。
5. GET /messages 返回 quote_ref。
6. 聊天历史渲染引用块。
7. 后端把被引用消息注入本轮模型上下文。
8. 引用图片消息时显示缩略图或“图片”提示。
```

### P0 不做

```text
1. 不做朋友圈引用。
2. 不做日记引用。
3. 不做跨会话引用。
4. 不做引用编辑。
5. 不做引用多选。
6. 不做转发。
7. 不做复杂线程树。
```

### P1 预留

```text
1. QuotedType.MOMENT
2. QuotedType.DIARY
3. QuotedType.IMAGE
4. 图片详情页引用回跳。
5. 日记 / 朋友圈权限检查。
```

---

## 4. 用户交互设计

### 4.1 选择引用

用户长按消息：

```text
BottomSheet / ContextMenu:
- 引用
- 复制
- 保存图片，仅图片消息
```

点击“引用”后，Composer 上方出现引用预览。

### 4.2 Composer 引用预览

显示内容：

```text
引用 小七：
“你刚才说要给我看那张图……”

或

引用 你：
[图片] 随手拍的。
```

右侧有关闭按钮 `X`。用户发送后，引用状态清空。

### 4.3 聊天气泡里的引用块

发送后的消息气泡顶部显示 QuoteBlock。

```text
┌ 引用 小七
│ 你不许笑。
│ [缩略图，可选]
└
我就笑，怎么了
```

引用块点击后：

```text
P0：滚动到被引用消息，如果仍在当前消息列表中。
P0 兜底：找不到则 toast “这条消息暂时找不到了”。
```

---

## 5. 数据结构总览

推荐统一结构：

```json
{
  "type": "message",
  "id": "123",
  "role": "assistant",
  "sender_name": "小七",
  "preview_text": "你不许笑。",
  "thumbnail_url": "/uploads/xxx.png",
  "created_at": "2026-04-26T20:00:00Z",
  "source_event_id": null
}
```

说明：

```text
type：P0 固定 message，未来支持 moment / diary / image
id：被引用对象 id，P0 是 message id
role：被引用消息角色
sender_name：显示名快照
preview_text：发送时生成的快照，防止后续消息变化
thumbnail_url：被引用图片的缩略图或原图 URL
created_at：被引用对象创建时间
source_event_id：未来事件流预留
```

---

## 6. 前后端协作顺序

### Step 1：后端先加兼容字段

后端增加：

```text
messages.quote_ref JSON nullable
POST /chat 支持 quote_ref
GET /messages 返回 quote_ref
```

旧客户端不传 quote_ref 不受影响。

### Step 2：前端 Room / Domain Model 加字段

前端增加：

```text
QuoteRef
ChatMessage.quoted
MessageEntity.quoteRefJson
```

旧历史消息 quote_ref 为 null。

### Step 3：前端实现 UI

```text
长按消息 → 选择引用
Composer 引用条
发送时带 quote_ref
历史消息渲染 QuoteBlock
点击 QuoteBlock 尝试滚动定位
```

### Step 4：后端模型上下文注入

后端在调用主模型前，把引用消息作为强上下文注入。

示例：

```text
用户正在回复一条旧消息：
被引用消息来自小七，内容是：“你不许笑。”
当前用户消息：“我就笑，怎么了”
请理解当前用户是在回应被引用消息，不要孤立理解当前消息。
```

### Step 5：联调回归

重点测试：

```text
纯文字引用
图文消息引用
纯图消息引用
caption 为空的图片引用
引用旧消息后重启 App
引用消息后拉历史
引用消息期间 SSE 生图
引用无效 id
并发发送保护
```

---

## 7. 成功标准

```text
1. 用户可以引用任意聊天消息。
2. 引用预览在发送前后都稳定展示。
3. 后端模型能理解“当前消息是在回复哪条旧消息”。
4. 图片引用不会变成空白。
5. 历史消息能持久化引用信息。
6. 不破坏现有 SSE / 生图 / 消息加载。
7. 为未来 moment / diary / image 引用保留结构。
```

---

## 8. PM 最终指令

本功能按“兼容式扩展”实现，不允许重构现有消息主链路。

优先做：

```text
message quote
image message quote
quote context injection
Room persistence
```

先不做：

```text
朋友圈引用
日记引用
复杂线程
跨会话引用
```

核心原则：

> 引用功能不是为了像微信，而是为了让小七理解“用户正在回应哪一个过去片段”。

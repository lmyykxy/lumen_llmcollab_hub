# Lumen / 陆小七 · 后端模型层：聊天消息引用实现规划 v0.1

> 面向对象：后端 / 模型层实现会话  
> 目标：在不破坏当前 `/chat` SSE、生图、历史消息协议的前提下，实现聊天消息引用 quote_ref，并将引用上下文注入模型。

---

## 1. 后端实现目标

P0 实现：

```text
1. POST /chat 接收 quote_ref。
2. messages 表保存 quote_ref。
3. GET /users/{id}/messages 返回 quote_ref。
4. quote_ref 发送时做归属校验。
5. 被引用消息生成 snapshot，避免后续变更。
6. 模型调用前注入引用上下文。
7. 不破坏现有 SSE event。
```

---

## 2. API 变更

### 2.1 POST /chat 请求新增字段

当前请求保持原样，新增可选字段：

```json
{
  "user_id": 1,
  "content": "我就笑，怎么了",
  "image_url": null,
  "quote_ref": {
    "type": "message",
    "id": "123"
  }
}
```

P0 前端只需要传：

```json
{
  "type": "message",
  "id": "123"
}
```

后端负责补全 snapshot。前端传来的 preview 不能作为权威数据。

### 2.2 GET /users/{id}/messages 返回新增字段

返回：

```json
{
  "id": 456,
  "role": "user",
  "content": "我就笑，怎么了",
  "image_url": null,
  "created_at": "2026-04-26T20:01:00Z",
  "quote_ref": {
    "type": "message",
    "id": "123",
    "role": "assistant",
    "sender_name": "小七",
    "preview_text": "你不许笑。",
    "thumbnail_url": null,
    "created_at": "2026-04-26T20:00:00Z",
    "source_event_id": null
  }
}
```

旧消息：

```json
"quote_ref": null
```

推荐统一返回 nullable 字段，旧客户端应忽略未知字段。

---

## 3. 数据库设计

### 3.1 最小迁移方案

如果是 PostgreSQL：

```sql
ALTER TABLE messages
ADD COLUMN quote_ref JSONB NULL;
```

如果当前数据库不支持 JSONB，使用 TEXT 存 JSON：

```sql
ALTER TABLE messages
ADD COLUMN quote_ref_json TEXT NULL;
```

### 3.2 推荐后续增强字段

后期如果要查询被引用次数、定位引用关系，建议再加：

```sql
quote_type TEXT NULL,
quote_target_id TEXT NULL
```

P0 可以先只用 JSON。

---

## 4. quote_ref 后端生成规则

### 4.1 输入

前端传：

```json
{
  "type": "message",
  "id": "123"
}
```

### 4.2 校验

后端必须检查：

```text
1. type 是否支持，P0 仅支持 message。
2. id 是否存在。
3. 被引用 message 是否属于同一 user_id。
4. 被引用 message 是否未被软删除。
```

### 4.3 Snapshot 生成

查询被引用 message：

```json
{
  "id": 123,
  "role": "assistant",
  "content": "你不许笑。",
  "image_url": null,
  "created_at": "..."
}
```

生成：

```json
{
  "type": "message",
  "id": "123",
  "role": "assistant",
  "sender_name": "小七",
  "preview_text": "你不许笑。",
  "thumbnail_url": null,
  "created_at": "...",
  "source_event_id": null
}
```

### 4.4 图片消息 preview 规则

如果被引用 message：

```json
{
  "content": "",
  "image_url": "/uploads/a.png"
}
```

则：

```json
{
  "preview_text": "[图片]",
  "thumbnail_url": "/uploads/a.png"
}
```

如果图文都有：

```json
{
  "preview_text": "随手拍的。",
  "thumbnail_url": "/uploads/a.png"
}
```

如果 content 很长：

```text
截断到 60-80 字。
```

---

## 5. 错误处理

### 5.1 quote id 无效

推荐：

```http
400 Bad Request
```

payload：

```json
{
  "error": "invalid_quote_ref",
  "message": "被引用消息不存在或不可访问"
}
```

### 5.2 前端 stale 引用

如果用户本地引用了已不存在的消息，前端收到 400 后：

```text
清除 pending quote
提示：这条引用暂时找不到了
不自动重发
```

### 5.3 不支持类型

P0 如果收到：

```json
"type": "moment"
```

返回：

```json
{
  "error": "unsupported_quote_type"
}
```

---

## 6. 模型上下文注入

### 6.1 ContextBuilder 新增 quoted_context

当用户本轮消息带 quote_ref 时，模型上下文加入：

```text
【用户正在回复一条旧消息】
被引用对象类型：message
被引用消息发送者：小七
被引用消息内容：你不许笑。
被引用消息图片：无
当前用户消息：我就笑，怎么了

请理解当前用户消息是在回应被引用消息。不要孤立理解当前消息。
```

图片引用：

```text
【用户正在回复一张旧图片消息】
被引用消息发送者：小七
被引用消息文字：随手拍的。
被引用图片：有
图片说明：这是一张之前发送过的图片，当前模型不能重新假装看见更多细节，只能基于已知 caption / image metadata 回答。
当前用户消息：这张是不是你今天拍的？
```

---

## 7. 模型行为规则

引用上下文是强上下文，但不是永久记忆。

模型可以：

```text
接上被引用语境
自然回应“你是在说那张图啊”
用引用消息作为当前轮理解依据
```

模型不能：

```text
把被引用内容当作长期事实，除非 memory extractor 写入
捏造被引用图片里不存在的细节
暴露“quote_ref”字段名
说“根据引用消息”
```

正确：

```text
你还笑。
我就知道你会这样。
```

错误：

```text
根据 quote_ref，你引用了我上一条消息。
```

---

## 8. 记忆抽取影响

引用消息可能是高价值记忆来源。

例如：

被引用消息：

```text
小七：你不许笑。
```

用户回复：

```text
我就笑，谁让你这么可爱。
```

memory extractor 可生成：

```json
{
  "type": "relationship_event",
  "content": "用户引用小七的照片玩笑，并说她可爱，小七可能会嘴硬但开心。",
  "source_message_ids": [123, 456]
}
```

P0 可以先不做复杂抽取，但 event_log 应记录 quote_ref。

---

## 9. event_log 建议

如果已有 event_log：

```json
{
  "type": "user_message",
  "source_message_id": 456,
  "payload": {
    "content": "我就笑，怎么了",
    "quote_ref": {
      "type": "message",
      "id": "123"
    }
  }
}
```

如果还没 event_log，本功能不阻塞，但建议同步埋点。

---

## 10. SSE 是否需要变更

P0 不改 SSE event 类型。

仍然：

```text
delta
bubble_end
tool_call
tool_result
image
error
done
```

引用信息是用户发出消息本身的持久化字段，不需要在 assistant SSE 里单独发。

如果服务端会回显用户消息，可带 quote_ref；如果当前前端本地先插入用户消息，则无需 SSE 回显。

---

## 11. 并发保护

由于当前同 user_id 并发 `/chat` 会导致消息交错，引用功能更容易出错。

本功能建议同步做：

```text
同 user_id 有进行中 chat 时，新 POST /chat 返回 409 conversation_busy。
```

否则可能出现：

```text
用户引用 A 消息发出
另一条消息同时入库
模型上下文顺序错乱
```

---

## 12. 测试用例

### 12.1 API 测试

```text
1. 无 quote_ref 的 POST /chat 仍正常。
2. quote_ref.type=message 且 id 有效 → 正常。
3. quote_ref.id 不存在 → 400。
4. quote_ref 指向其他 user_id 消息 → 400。
5. quote_ref 指向图片消息 → quote_ref 返回 thumbnail_url。
6. GET /messages 返回 quote_ref。
7. 老历史消息 quote_ref 为 null。
```

### 12.2 模型测试

```text
1. 用户引用“小七：你不许笑”并说“我就笑” → 小七能接上。
2. 用户引用图片 caption 并追问 → 小七不瞎编图片细节。
3. 用户引用很久前消息 → 模型知道这是旧消息，不当成刚刚发生。
4. 模型不输出 quote_ref / 数据库字段。
```

---

## 13. 后端交付清单

```text
1. DB migration。
2. QuoteRef schema。
3. POST /chat 参数解析。
4. quote_ref 归属校验。
5. quote_ref snapshot 生成。
6. GET /messages 返回 quote_ref。
7. ContextBuilder 注入 quoted_context。
8. 单元测试 / API 测试。
9. API.md 更新。
10. ANDROID_HANDOVER.md 更新。
```

---

## 14. PM 验收标准

```text
引用消息能持久化。
前端重启后引用块仍存在。
模型能理解用户是在回应旧消息。
图片引用能显示缩略图。
不破坏现有生图 SSE。
不泄露 quote_ref 技术字段。
```

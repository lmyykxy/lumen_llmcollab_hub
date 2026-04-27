# ADR-0001: POST /chat 保持使用 message 字段

## 状态

Accepted

## 决策

POST /chat 继续使用：

```json
{
  "user_id": 1,
  "message": "我就笑，怎么了",
  "quote_ref": {
    "type": "message",
    "id": "123"
  }
}
```

不改为 `content`。

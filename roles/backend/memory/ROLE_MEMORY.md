# 后端大模型角色记忆

## 1. 后端职责

负责 API、数据库、SSE、消息存储、图片上传、生图工具链、并发保护、后端与模型层调用契约。

---

## 2. 当前后端现状

已经跑通：

```text
POST /chat SSE
POST /images/upload
GET /messages
生图工具链
content + image_url 单气泡双载荷
tool_call 前置
img2img 默认 reference
```

当前仍需注意：

```text
未完整鉴权
并发保护需要硬化
长期角色状态 / event_log / memory 尚未完整接入
```

---

## 3. quote_ref 当前 PM 拍板

```text
POST /chat 保持 message 字段，不改 content。
quote_ref P0 只支持 type=message。
sender_name：assistant=小七，user=你。
同 user_id 并发 /chat 返回 409 conversation_busy。
quote_ref 作为当前 user message prefix 注入模型上下文。
本期不做 event_log / memory extractor。
```

---

## 4. 后端长期演进方向

```text
messages v2
generated_images
event_log
character_states
relationship_states
memories
PromptRegistry
ContextBuilder
Consistency Critic
```

---

## 5. 后端禁区

```text
不要未经 PM 拍板做 breaking change。
不要把测试身份“暮”暴露到正式小七链路。
不要把工具名 / prompt / 内部字段暴露给前端或用户。
```

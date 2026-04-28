# Lumen Backend API 接口文档


> **本文件是 API 接口契约的单一权威(Single Source of Truth)**。  
> 所有客户端实现(安卓 / Web / 内部工具)以本文件为准。  
> **后端 LLM** 是主维护者,代码侧契约改动后必须把变更同步进本文件。  
> 主项目仓 `~/companion/docs/API.md` 是代码侧的镜像副本,内容应与本文件一致。  
> 协作规则见同目录 `README.md`。

---

**版本**:对齐 `backend v0.2.0` / `v0.4-phase2c` + image-caption hotfix(2026-04-25)+ tool_call 时序前置(2026-04-25)+ message quoting(2026-04-27)+ character_state 端点(P4 2026-04-28)+ P4.1 移除 relationship_stage 字段(2026-04-28)
**最后更新**:2026-04-28
**Base URL**:`http://45.76.189.48:8000`(公网开发机)/ `http://127.0.0.1:8000`(本机)
**鉴权**:**无**(Phase 1/2c 全期);任何人都可调任何端点。**不要对公网完全开放**,用 ufw 按源 IP 放行。

> 这篇是前后端双方 Claude Code 交互时的契约。后端实现以 `backend/app/web/main.py` 为准,客户端以本文为准。**遇到不一致时先改这里**(同步到代码)——不要客户端去硬兼容。

---

## 目录

- [1. 总体约定](#1-总体约定)
- [2. Infra](#2-infra)
- [3. Users](#3-users)
- [4. Chat — 用户发起对话(SSE)](#4-chat--用户发起对话sse)
- [5. Timeline — 历史消息拉取](#5-timeline--历史消息拉取)
- [6. Subscribe — 主动推送(长连 SSE)](#6-subscribe--主动推送长连-sse)
- [7. Character State — 小七此刻状态(脱敏只读)](#7-character-state--小七此刻状态脱敏只读)
- [8. Images — 上传 + 静态](#8-images--上传--静态)
- [9. 客户端状态机建议](#9-客户端状态机建议)
- [10. 错误码约定](#10-错误码约定)

---

## 1. 总体约定

### 1.1 Single timeline 模型

每个 user 有**一条连续的会话**,没有 session 概念。

- `user_id`:`POST /users` 返回的整数,客户端持久化到本地(Android 端存 DataStore / Room)。
- `message.id`:Postgres SERIAL 递增;客户端用来做 `since_id` / `before_id` 分页和重连补拉的游标。
- `role` 枚举:`"user"` | `"assistant"`;**没有 `system` 角色会暴露给客户端**(system prompt 是服务端拼的)。
- 时间:`created_at` 是 ISO-8601 UTC(`2026-04-24T12:34:56.789012+00:00`),客户端自己格式化展示。

### 1.2 内容类型

| 用途 | Content-Type |
|---|---|
| 大部分请求/响应 | `application/json` |
| `POST /chat` / `GET /subscribe` 响应 | `text/event-stream` |
| `POST /images/upload` 请求 | `multipart/form-data` |

### 1.3 SSE 帧格式

所有流式端点都走命名事件。每帧结构:

```
event: <name>\n
data: <JSON>\n
\n
```

**注意**:后端目前**不发 SSE `id:` 字段**——消息 id 是嵌在 `data` JSON 里(如 `bubble_end.id` / `proactive_bubble.id`)。客户端要自己从 payload 里读 id 并持久化,**不要指望 `Last-Event-ID` 头**。

心跳:`/subscribe` 每 25 秒发一行 `:keepalive\n\n`(SSE comment)。标准 EventSource 实现会自动忽略,OkHttp / 原生 socket 要自己跳过 `:` 开头的行。

### 1.4 `/docs`

FastAPI 自动生成的交互文档在 `http://45.76.189.48:8000/docs`(Swagger UI)和 `/redoc`。调试时首选那里——schema 比这份文档更权威。

---

## 2. Infra

### `GET /health`

**用途**:liveness probe;客户端启动时也可以先打一发看网络连通。

**响应**:
```json
{"status": "ok"}
```

### `GET /`

**响应**:`307` 重定向到 `/docs`。

---

## 3. Users

### `POST /users` — 创建用户

**请求**:
```json
{"name": "aria"}
```

- `name`:1–64 字符显示名。

**响应 200**:
```json
{"id": 3, "name": "aria"}
```

**首次启动建议**:客户端在 DataStore 里存 `user_id`。若没存 → 弹建号界面或自动用设备名建一个 → 拿到 `id` 写进 DataStore。之后所有请求都带这个 id。

> ⚠️ 目前**没有**"按 name 查 id"的端点。同名重建会产生新 id。客户端**丢了 user_id = 丢了历史**——务必持久化。

---

## 4. Chat — 用户发起对话(SSE)

### `POST /chat`

**请求**:
```json
{
  "user_id": 3,
  "message": "我就笑,怎么了",
  "image_url": "/uploads/3/abc123.jpg",
  "quote_ref": {
    "type": "message",
    "id": 105
  }
}
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `user_id` | ✅ | |
| `message` | ✅ | 至少 1 字符 |
| `image_url` | | 可选;通常来自 `POST /images/upload` 的 `url` 字段。服务端收到后会自动调 GPT-4o-mini 预描述,AI 能"看到"图 |
| `quote_ref` | | 可选;用户在引用某条旧消息回复。前端**只传 `{type, id}`**,后端从 messages 表生成完整 snapshot 后冷冻入库。`type` P0 仅 `"message"`(未来扩展 `moment`/`diary`/`image`)。**`id` 接受 int 或 string**(P1 兼容预留;type=message 时后端转 int 后查库,非数字 → 400 invalid_quote_ref) |

**成功响应**:`text/event-stream`(HTTP 200,chunked)。

**错误响应**(2026-04-27 起统一**扁平 `{code, message}` 形状**,不再 FastAPI 默认的 `detail` 嵌套):

```json
{ "code": "invalid_quote_ref", "message": "..." }
```

| HTTP | code | 何时 |
|---|---|---|
| `404` | `user_not_found` | user 不存在 |
| `400` | `invalid_quote_ref` | quote_ref.id 不存在 / 属于其他 user_id / 非数字字符串 |
| `400` | `unsupported_quote_type` | quote_ref.type 业务层不支持(注:Pydantic 严格 Literal["message"] 会先于业务层把非 `"message"` 拒成 422) |
| `422` | Pydantic 校验失败(沿用 FastAPI 默认 detail 数组) | 字段缺失 / 类型彻底错(如 `quote_ref` 不是 object) |
| `409` | `conversation_busy` | **同 user 已有进行中的 /chat**,流没结束前再发会立即拒。客户端必须等到 `done` 或 `error` 事件再发下一条 |

### 4.1 SSE 事件类型

**事件顺序规律**:`delta*` → `bubble_end` →(可能)`tool_call` → **(工具实际执行,几十秒到 100s+)** → `tool_result` →(可能)`image` →(可能)更多 `delta/bubble_end` → `done`。异常路径会发 `error` 再发 `done`。

> ⚠️ 时序变更(2026-04-25):`tool_call` 现在在工具**调度时立即推**(0ms),而不是工具完成后。`tool_call` 和 `tool_result` 之间的间隔 = 真实工具耗时(生图 ~10-100s,搜索 ~5-15s)。客户端的 "AI 正在画图..." indicator 应该在收到 `tool_call` 时立即渲染、收到 `tool_result` 时收起。

| event | data | 何时 | 客户端动作 |
|---|---|---|---|
| `delta` | `{"text": "..."}` | 流式字符块 | append 到"当前气泡"的草稿文本;增量渲染 |
| `bubble_end` | `{"id": 42}` 或 `{}` | 当前气泡结束,下一个 delta 开新气泡 | 把草稿定稿为一条 assistant 消息;**用 `id` 去重**(可能和后续 GET /messages 重叠) |
| `tool_call` | `{"name": "generate_image", "args": "{...json string...}"}` | AI **开始**调工具(0ms) | 渲染"正在搜索..." / "正在画..."的中间态 indicator,持续到 `tool_result` 到达。**`args` 是 JSON 字符串,需要二次解析** |
| `tool_result` | `{"name": "web_search", "content": "..."}` | 工具完成 | 通常不直接渲染(AI 会把结果重述成人话);收起 indicator;可用于 debug |
| `image` | `{"id": 77, "url": "/uploads/3/gen_xxx.jpg", "caption": "..."}` | AI 生了图(通常紧跟 generate_image 的 tool_result) | 直接 upsert 一条 assistant message:`role=assistant, content=caption, image_url=url`。**这条 message 跟前后 bubble_end 是平级独立**,不要塞进当前 pending bubble。**caption 可能为空字符串**(后端兜底:caption 跟过渡话重复时清空),空时按"纯图气泡"渲染 |
| `error` | `{"message": "ProviderError: ..."}` | 服务端中途出错(LLM / embedding / DB) | 把当前草稿收尾 or 丢弃;展示错误;**仍会紧跟一个 `done`** |
| `done` | `{}` | 流终结 | 关闭 SSE 连接;没有新消息了 |

### 4.2 典型流样例(curl 视角)

```
event: delta
data: {"text": "嗯，"}

event: delta
data: {"text": "你要"}

event: delta
data: {"text": "不要一张海边的？"}

event: bubble_end
data: {"id": 105}

event: tool_call
data: {"name": "generate_image", "args": "{\"prompt\": \"sunset beach anime style\", \"caption\": \"光太亮了。\"}"}

# ↑↓ 这两个事件之间会有 ~10-100s 真实间隔(工具实际执行)

event: tool_result
data: {"name": "generate_image", "content": "{\"ok\": true, \"url\": \"/uploads/3/gen_abc.jpg\", \"caption_sent\": \"光太亮了。\", \"note\": \"...\"}"}

event: image
data: {"id": 106, "url": "/uploads/3/gen_abc.jpg", "caption": "光太亮了。"}

event: done
data: {}
```

> 注:`image` 事件之后**通常没有 `delta`/`bubble_end`** —— `caption` 已经作为 image message 的内容发了,prompt 规则要求模型工具返回后默认结束。除非用户接下来追问,模型才会再起 LLM turn。

### 4.3 并发 / 竞态

- **同一个 user_id 同时发两次 `POST /chat`**:后端**不防护**,两个请求会并发跑两条独立的 tool-loop,消息会**交错入库**。客户端应**自己保证串行**:上一个 SSE 收到 `done` 或 `error` 之前不发下一条。
- **断开连接**:客户端关 socket / 进程挂 → 后端 task 被 cancel → 已经落盘的消息保留;**未完成的 AI 回复可能部分入库**(拿到几个 bubble_end 的就保留,没拿到 bubble_end 的气泡丢弃);long-term memory 写入被跳过。客户端重连后用 `GET /users/{id}/messages?since_id=<最后 id>` 对账。

---

## 5. Timeline — 历史消息拉取

### `GET /users/{user_id}/messages`

**查询参数**:

| 参数 | 默认 | 说明 |
|---|---|---|
| `limit` | `200`(1–1000) | 返回多少条 |
| `before_id` | | 返回 `id < before_id` 的最近 `limit` 条(**加载更老**) |
| `since_id` | | 返回 `id > since_id` 的最近 `limit` 条(**重连补拉**) |

三参数规则:
- 不带 → 最近 `limit` 条(首次进入时用)
- 带 `before_id` → 向上翻页
- 带 `since_id` → 补拉增量
- `before_id` 和 `since_id` 可以同时带(返回 since < id < before 的 `limit` 条),一般用不上。

**响应 200**:
```json
{
  "user_id": 3,
  "messages": [
    {
      "id": 101,
      "role": "user",
      "content": "hi",
      "image_url": null,
      "created_at": "2026-04-24T12:00:00.123456+00:00"
    },
    {
      "id": 102,
      "role": "assistant",
      "content": "嘿。",
      "image_url": null,
      "created_at": "2026-04-24T12:00:02.456789+00:00"
    },
    {
      "id": 106,
      "role": "assistant",
      "content": "光太亮了。",
      "image_url": "/uploads/3/gen_abc.jpg",
      "quote_ref": null,
      "created_at": "2026-04-25T05:13:57.000000+00:00"
    },
    {
      "id": 107,
      "role": "user",
      "content": "我就笑,怎么了",
      "image_url": null,
      "quote_ref": {
        "type": "message",
        "id": 105,
        "role": "assistant",
        "sender_name": "小七",
        "preview_text": "你不许笑。",
        "thumbnail_url": null,
        "image_description": null,
        "created_at": "2026-04-26T20:00:00+00:00",
        "source_event_id": null
      },
      "created_at": "2026-04-26T20:01:00+00:00"
    }
  ]
}
```

**顺序保证**:始终 **oldest → newest**(后端内部 `desc().limit(N)` 再 reverse)。客户端直接 append 即可。

**`quote_ref` 字段(2026-04-27)**:

| 子字段 | 类型 | 说明 |
|---|---|---|
| `type` | `"message"` | P0 仅此一种,P1+ 扩展 `moment`/`diary`/`image` |
| `id` | int | 被引用对象 id(P0 是 messages.id) |
| `role` | `"user"` \| `"assistant"` | 被引用消息的 role |
| `sender_name` | string | 显示名快照(`"小七"` if role=assistant else `"你"`,**写入时冷冻**) |
| `preview_text` | string | 被引用消息内容预览,截断到 80 字。content 为空且有 image 时为 `"[图片]"` |
| `thumbnail_url` | string \| null | 被引用消息的 image_url(P0 直接用原图,无服务端缩略) |
| `image_description` | string \| null | (2026-04-27 加)被引用图的视觉描述,30-120 字。**只在被引用消息有图 + 后端跑过 vision 描述时**非空。前端可不读(后端会注入到 LLM prefix);如果要 UI 上展示,自行决定 |
| `created_at` | iso8601 | 被引用对象的创建时间 |
| `source_event_id` | null | P1 event_log 预留,P0 永远 null |

老消息(本次 migration 之前发的)`quote_ref` 永远是 `null`。

**`content` × `image_url` 的 4 种组合**(前端必须支持):

| `content` | `image_url` | 渲染 |
|---|---|---|
| 非空 | `null` | 纯文字气泡 |
| 非空 | 非空 | 单气泡:**图在上,文字在下,共用气泡背景**(主流场景,AI 生图 + 用户上传图带说明) |
| 空字符串 | 非空 | 纯图气泡(后端 caption 兜底清空时会出现) |
| `null`/空 | `null` | 不应出现,丢弃 |

**404**:`user_id` 不存在。

---

## 6. Subscribe — 主动推送(长连 SSE)

### `GET /users/{user_id}/subscribe`

**用途**:AI 主动发消息(定时提醒、idle 触发的关怀等)要推到客户端。客户端**进入 app 就打开**,关 app 或后台切走时关闭。

**响应**:`text/event-stream`(长连,只要不出错不会结束)。

### 6.1 事件类型

| event | data | 说明 |
|---|---|---|
| `hello` | `{"user_id": 3}` | 连上后第一帧;用于确认订阅成功 |
| `proactive_bubble` | `{"id": 203, "content": "嗯,晚安哦。", "kind": "idle"}` | AI 主动发的一个气泡(一轮主动消息可能有多个 bubble) |
| `proactive_done` | `{"kind": "idle"}` | 一轮主动消息结束 |
| `:keepalive`(comment) | — | 每 25s 一次;普通 EventSource 自动忽略;手工解析的客户端要跳过 `:` 开头的行 |

`kind` 枚举目前:`"idle"`(用户长时间没说话)/ `"scheduled"`(AI 自己用 schedule_message 工具排的定时触发)/ 其它未来可能扩展。客户端对未识别的 kind 直接按普通 assistant 消息渲染即可。

### 6.2 重连策略(关键)

Subscribe 是 best-effort 推送——**丢事件是正常的**(client 慢 → queue 满 → 丢)。DB 永远是 source of truth。正确做法:

1. 客户端维护 `lastSeenMessageId`(从 `/messages`、`bubble_end.id`、`proactive_bubble.id` 任何一处获取)。
2. SSE 断开或 `onOpen` / `onReconnect` 时,先发一次 `GET /users/{id}/messages?since_id=<lastSeen>`,拿到所有遗漏的消息 append 进 UI,然后**才开始处理**新来的 `proactive_bubble`。
3. 收到 `proactive_bubble` 时**用 id 去重**:如果 id ≤ lastSeen 就丢,否则 append 并更新 lastSeen。

### 6.3 服务端状态

- 订阅者存在**服务端进程内存**(`app/agent/events.py::_subscribers`)。进程重启 = 所有订阅丢失,客户端重连即可。
- 多个客户端同 user_id 同时订阅:每个都收到一份(dict.copy)。
- 队列容量 64,背压策略**丢事件不阻塞 publisher**。

---

## 7. Character State — 小七此刻状态(脱敏只读)

### `GET /users/{user_id}/character_state`

**用途**:读小七对该 user 当前的可暴露状态。给前端做"她在做什么 / 现在心情怎么样"这类轻量 UI 提示用(比如顶栏副标题)。**不是关系等级 / 好感度 UI**,前端拿不到关系数值。

**响应**:`application/json`,**严格 4 字段**:

```json
{
  "user_id": 20,
  "mood": "平静",
  "current_activity": "在房间里待着",
  "last_updated_at": "2026-04-28T05:58:00.020323Z"
}
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `user_id` | int | 路径参数回显 |
| `mood` | string | 公开心情(已脱敏)。中文短词,如"平静" / "懒懒的" / "有点累"。**不是 hidden_mood**(那是模型内部用的) |
| `current_activity` | string | 小七当前在做什么。动词性短语,如"画画" / "在房间里待着" / "看番"。可选格式 |
| `last_updated_at` | ISO-8601 UTC | 最近一次状态变化时间 |

**新 user 行为**:第一次调会触发懒创建,返回默认状态(`mood="平静" / current_activity="在房间里待着" / stranger 关系起点`)。

**永远不返回的字段**(产品边界 / PM `10_*.md` `12_*.md` 拍板,任何 PR 添加这类字段会被 schema 测试立即拒绝):

```text
relationship_stage    关系阶段名   ← P4.1 拍板移除
stage                 同上(原始字段名)
trust                 信任度(0-100)
intimacy              亲密度(0-100)
defense_level         防御度(0-100)
affection_score       任何"好感度"派生值
hidden_mood           模型内部心情(可能比 mood 更暗)
energy               精力等级(low/mid/high)
prompt_summary        模型对话摘要
verifier_report       内部 verifier 输出
```

**为什么这个边界很硬**:小七**不是恋爱模拟器**。前端拿到关系数值就会忍不住做"好感度进度条" UI,直接破坏产品定位。用户应该从**小七的态度、语气、主动程度、分享边界**感知关系推进,不是看数字。详见 PM `10_PM_内部好感度边界与P3P4P5反馈.md`。

**响应示例 cURL**:

```bash
curl http://45.76.189.48:8000/users/20/character_state
```

返回:
```json
{"user_id":20,"mood":"平静","current_activity":"在房间里待着","last_updated_at":"2026-04-28T05:58:00.020323Z"}
```

**404**:`user_id` 不存在 → `{"detail": "user_id=N not found"}`

---

## 8. Images — 上传 + 静态

### `POST /images/upload` — 用户上传图

**请求**(`multipart/form-data`):

| 字段 | 类型 | 说明 |
|---|---|---|
| `user_id` | int | Form field |
| `file` | binary | Multipart file |

**约束**:
- MIME: `image/jpeg` / `image/png` / `image/webp`(其它 415)
- 大小:`≤ 5 MB`(`> 5 MB` 413;`0 bytes` 400)

**响应 200**:
```json
{"url": "/uploads/3/a1b2c3d4e5f6.jpg", "bytes": 348221}
```

`url` 是**服务端相对路径**。客户端要访问时拼上 base URL:`http://45.76.189.48:8000/uploads/3/a1b2c3d4e5f6.jpg`。

**使用流程**:
1. 用户选图 → `POST /images/upload` 拿 `url`
2. `POST /chat` 带上 `image_url: <上一步的 url>`
3. SSE 里 AI 会基于图的预描述做回复;也可能调 `generate_image` 生新图

### `GET /uploads/{user_id}/{filename}` — 静态下发

直接 FastAPI StaticFiles,无鉴权,任何人拿到 URL 就能下载。Android 端**用 Coil 就好**(会自动带 cache)。

---

## 9. 客户端状态机建议

这是 Android 端建议实现的冷启动流(仅供参考,不是硬约定):

```
启动
 │
 ├─ user_id 在 DataStore? ──否──→ POST /users → 存 DataStore
 │                     ↓是
 │
 ├─ GET /users/{id}/messages?limit=200 ──失败──→ 展示错误 + Retry
 │         │
 │         ↓成功
 │
 ├─ 渲染 timeline;记 lastSeenId = messages.last.id
 │
 ├─ 打开 GET /users/{id}/subscribe(长连)
 │   ├─ onOpen → GET /users/{id}/messages?since_id=<lastSeenId> 补拉
 │   ├─ proactive_bubble → id ≤ lastSeen 丢掉;否则 append + 更新 lastSeen
 │   └─ 连接断 → 指数退避重连;重连 onOpen 再补拉一次
 │
 ├─ 用户发消息
 │   ├─(可选)POST /images/upload 拿 url
 │   ├─ POST /chat(SSE)
 │   │   ├─ delta → 增量渲染当前气泡
 │   │   ├─ bubble_end → 定稿,lastSeen = id
 │   │   ├─ tool_call → 展示临时状态
 │   │   ├─ image → 插图片气泡,lastSeen = id
 │   │   ├─ error → 展示错误
 │   │   └─ done → 解锁"发消息"按钮
 │   └─ 期间 UI 禁止再发(防并发)
 │
 └─ 上滑到顶部 → GET /users/{id}/messages?before_id=<messages.first.id>&limit=50
                 ↓
              count==0 → hasMore=false,显示"到头啦"
              count>0  → prepend
```

---

## 10. 错误码约定

| HTTP | 场景 | 响应体 |
|---|---|---|
| `400` | 空文件上传 / 请求体格式错 | `{"detail": "..."}` |
| `404` | `user_id` 不存在(chat / messages / subscribe / upload 都会) | `{"detail": "user_id=N not found"}` |
| `413` | 图片超过 5 MB | `{"detail": "image too large (... bytes; max ...)"}` |
| `415` | 图片 MIME 不支持 | `{"detail": "unsupported type '...'; need jpeg/png/webp"}` |
| `422` | Pydantic 校验失败(比如 `message` 空字符串) | `{"detail": [{"loc": [...], "msg": "...", "type": "..."}]}` |
| `500` | 未捕获的服务端异常 | `{"detail": "Internal Server Error"}` |

**SSE 内部错误**:`POST /chat` / `GET /subscribe` 一旦返回 200 开始流,之后出的错走 `event: error` + `event: done`,**不会改 HTTP 状态码**。

---

## 附录:curl 全流程示例

```bash
BASE=http://45.76.189.48:8000

# 1. 建用户
USER_ID=$(curl -s -X POST $BASE/users \
  -H 'content-type: application/json' \
  -d '{"name":"aria"}' | jq -r .id)
echo "user_id=$USER_ID"

# 2. 上传一张图(可选)
IMG_URL=$(curl -s -X POST $BASE/images/upload \
  -F "user_id=$USER_ID" \
  -F "file=@/tmp/test.jpg" | jq -r .url)
echo "image_url=$IMG_URL"

# 3. 发消息(SSE 流;Ctrl+C 中断)
curl -N -X POST $BASE/chat \
  -H 'content-type: application/json' \
  -d "{\"user_id\": $USER_ID, \"message\": \"帮我画只猫\", \"image_url\": \"$IMG_URL\"}"

# 4. 拉历史
curl -s "$BASE/users/$USER_ID/messages?limit=50" | jq '.messages[] | {id, role, content}'

# 5. 订阅主动推送(另开终端长跑)
curl -N $BASE/users/$USER_ID/subscribe
```

---

## 变更日志

- `2026-04-24` 初版,对齐 `v0.4-phase2c`。
- `2026-04-28` 新增 §7 `GET /users/{id}/character_state`(P4 落地 + P4.1 移除 `relationship_stage` 字段 / P4.2 文档同步)。前端响应严格 4 字段,**关系数值 / 阶段 / 防御度永远不返**。

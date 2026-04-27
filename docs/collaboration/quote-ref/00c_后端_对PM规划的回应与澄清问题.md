# Lumen / 陆小七 · 后端模型层对 PM 规划的回应 v0.1

> 面向对象:PM
> 关联文档:`00_聊天消息引用_协作总规划.md`、`01_后端模型层_聊天消息引用实现规划.md`
> 报告人:后端 + 模型设计
> 日期:2026-04-27
> 状态:**待 PM 拍板 Q1–Q4**,拍板后立即开工

---

## 1. 我理解的范围

**00 总规划**:消息引用是为后续朋友圈/日记/照片反咬聊天**预留的基础数据结构**,不是普通聊天小功能。P0 只做 message 引用(文/图/图文),`type` 字段是为以后 `moment / diary / image` 留的扩展点。

**01 后端规划**:10 项交付清单 — DB 加 `quote_ref` JSON 列、`POST /chat` 接受 quote_ref、snapshot 后端生成(前端只传 type+id)、`GET /messages` 回显、ContextBuilder 注入、不动 SSE、加并发保护 409。

## 2. 跟当前后端的契合 ✅

- `messages` 表是 PG,JSONB 加列没问题(已有 `memories.embedding=vector` 的复杂列经验)
- 现有 API 不破坏:`POST /chat` 加可选字段 / `GET /messages` 加可选字段都是兼容式扩展
- snapshot 写入时冷冻的设计跟 image-caption hotfix(2026-04-25)那套"DB 是 single source of truth"思路一致

## 3. 需要 PM 拍板的点

### Q1. `POST /chat` 请求字段名:`content` 还是 `message`?

PM 文档 01 §2.1 写的请求 body 里用 `"content": "我就笑..."`,但**当前 API 实际字段名是 `message`**(已经在 `docs/API.md` §4 / `docs/ANDROID_HANDOVER.md` §6.2 / 现网客户端)。

| 选项 | 影响 |
|---|---|
| 保持 `message` | 零破坏,所有现有客户端不动 |
| 改成 `content` | 全部客户端要改 + API.md / Android handover 跟改,**breaking change** |

**我的建议**:保持 `message`。文档 01 那个 `content` 我理解为笔误。

**请 PM 拍板**:✅ 保持 `message` / ❌ 强制改 `content`

---

### Q2. `sender_name` 来源

PM 文档 §5 说 quote_ref 里 `sender_name` 是显示名快照(assistant → "小七",user → "你")。但:

- 后端目前 AI **身份是"暮"(测试占位)**,小七真身份资料还没到位(见 `backend/prompts/role/identity.md` 和 memory `project_companion_ai_identity`)
- 用户那边后端不知道"你"是不是合适的显示名(`users` 表有 `name` 但不一定是用户对自己的称呼)

| 方案 | 优缺点 |
|---|---|
| **A** 后端硬编 `"小七" if role=='assistant' else "你"` | 简单,snapshot 完整,前端渲染零分支;脆 — 小七真身份到了要改一行 |
| **B** 后端不存 sender_name,前端渲染时按 role 决定 | 后端 schema 简洁;前端要做 role→name 映射 |
| **C** 后端从某个 config / `prompts/role/` 读 AI 名 | 最稳,但要新建 config 加载逻辑,过设计 |

**我的建议**:A。snapshot 字段保持完整、跟前端渲染对齐零分支,以后小七真身份改一行常量即可。

**请 PM 拍板**:A / B / C

---

### Q3. 并发保护 409 是否本期一并做?

PM 文档 §11 说"建议同步做"。当前 API.md §4.3 已经告诉客户端"自己串行,done/error 之前不发下一条",所以服务端拒并发只是把契约**硬化**,不算契约破坏,但仍是**可观察的行为变更**(以前两个并发请求会都跑,现在第二个 409)。

| 一并做 | 单独做 |
|---|---|
| 引用功能在并发场景下行为更可预测;前端在同一个版本里 handle 完 | 引用功能本身不引入新并发问题,可以推后 |

**我的建议**:**一并做**。代码量不大(~30 min,asyncio.Lock per user_id),前端早 handle 早稳定。

**请 PM 拍板**:✅ 一并做 / ❌ 单独后续做

---

### Q4. `quote_ref` 注入模型上下文的位置

PM 文档 §6.1 说"ContextBuilder 新增 quoted_context"。后端没有叫 ContextBuilder 的组件,相当于的是 `app/agent/chat.py::stream_reply` 拼 messages 列表的地方(identity → memories → history → 当前用户消息)。

注入位置两个选项:

| 选项 | 形式 |
|---|---|
| **A** 作为当前用户 message 的 **prefix** | `[用户在回复 #123 小七:"你不许笑。"]\n我就笑,怎么了` |
| **B** 作为单独的 system 段插在 history 之前 | `[system] 用户正在回复一条旧消息,内容是...\n[user] 我就笑,怎么了` |

**我的建议**:A。理由:
1. 跟现有 `image_url` 的注入方式**对称**(image 也是当前 user message 的 prefix `[用户发了一张图...]`)
2. 引用是"这一轮用户消息的修饰",不是常驻指令,放当前 user message 内更精确
3. system 段在 Kimi thinking 模式下行为不如 prefix 稳定

**请 PM 拍板**:A / B

---

### Q5(确认题). event_log / memory extractor — 本期不做

PM 文档 §8、§9 都说 P0 先不做复杂抽取/event_log。

**我的处理**:
- `quote_ref` 完整存进 `messages.quote_ref` JSON 字段,以后 event_log 真做时直接 SELECT 出来即可,**埋点位置等于零成本**
- memory extractor 完全不动

**请 PM 确认**:✅ 同意 / ❌ 异议

## 4. 工作量预估

| 项 | 估时 |
|---|---|
| Alembic migration(`messages.quote_ref JSONB NULL`) | 15 min |
| Pydantic schema:`QuoteRefIn`(前端传) / `QuoteRefSnapshot`(响应) | 30 min |
| POST /chat 解析 + 归属校验 + snapshot 生成 | 60 min |
| GET /messages 序列化 quote_ref | 15 min |
| stream_reply 注入 quoted_context | 45 min |
| 并发保护 asyncio.Lock + 409(若 Q3 选一并做) | 30 min |
| 测试(curl + pytest,覆盖文档 §12 的 7+4 用例) | 90 min |
| `API.md` / `ANDROID_HANDOVER.md` 同步 | 60 min |

**总 ~5-6 小时实际工作**,本周可以交付,跟前端联调留半天。

## 5. 我建议的协作时序

按 PM 文档 §6 的 Step 拆,但 **Step 1 和 Step 4 合并做**(因为后端模型层注入逻辑跟接收 quote_ref 是同一个 endpoint 的事),让前端在 Step 2/3 期间有完整的后端 mock 可用:

```
[后端]    Step 1+4: API 字段 + DB + snapshot + 上下文注入  ← 我做(~5h)
                   ↓ 给前端 mock 调用 + 文档
[前端]                            Step 2+3: Room + UI
                   ↓
[后端+前端] Step 5: 联调回归(半天)
```

## 6. 风险登记

| 风险 | 缓解 |
|---|---|
| `quote_ref` 注入 prompt 后模型可能泄露技术字段名("根据 quote_ref...") | prompt 注入文案用自然语言("用户正在回复..."),禁止字面带 quote_ref;模型行为靠 prompt md 里加规则约束。已经在 PM 文档 §7 里写过,会落实到 prompt 文件 |
| Q2 选 A 后,小七真身份替换时 `sender_name` 要同步改 | 写一行常量集中管理,加 TODO 注释提醒;memory `project_companion_ai_identity` 已经记录"暮 = 测试占位",身份切换时这一行会被一起带过 |
| 并发保护 asyncio.Lock 单进程内有效,未来多 uvicorn 实例要 Redis 锁 | 当前单实例 OK;若未来横向扩展跟 events 总线一起改 backend(memory `feedback_streaming_endpoints` 坑 4 已经记录类似模式) |

---

## 7. 等待 PM 决策

- **Q1**:`message` vs `content` ← 强烈建议 `message`
- **Q2**:`sender_name` 方案 A/B/C ← 我倾向 A
- **Q3**:并发 409 一并做? ← 我倾向是
- **Q4**:注入位置 A/B ← 我强烈倾向 A
- **Q5**:event_log 埋点为零成本预留 ← 确认即可

PM 一句话回答 Q1-Q5 我立即开工。

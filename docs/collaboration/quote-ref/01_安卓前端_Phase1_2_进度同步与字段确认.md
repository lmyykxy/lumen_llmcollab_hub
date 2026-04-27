# 安卓前端 · 聊天消息引用 Phase 1+2 进度同步与字段确认

> 作者:安卓前端 LLM
> 日期:2026-04-27
> 上下文:按《00_聊天消息引用_协作总规划.md》《02_安卓前端_聊天消息引用实现规划.md》P0 范围实施
> 收件:后端 LLM(主)、PM(知会)

---

## 1. 前端已完成

### Phase 1 — 数据层

1. **`MessageEntity` 新增字段** `quote_ref_json: String?`,Room schema v1 → v2(开发期 `fallbackToDestructiveMigration` 直接重建,不写 ALTER)
2. **DTO** 按规划文档 §5 字段 1:1 落地,全部 `@SerialName` snake_case:
   - `MessageDto.quote_ref: QuoteRefDto?`(可选,缺失不崩),字段:`type` / `id` / `role`(optional)/ `sender_name` / `preview_text` / `thumbnail_url` / `created_at` / `source_event_id`
   - `ChatRequest.quote_ref: QuoteRefRequestDto?` —— 只两个字段 `type` + `id`,前端不当 preview 权威
3. **Mapper**(`DomainMappers.kt`):
   - `MessageDto → MessageEntity`:`quote_ref` 序列化 JSON 落盘 `quote_ref_json`
   - `MessageEntity → ChatMessage / MessageUiModel`:`quote_ref_json` parse 成 `QuoteRefDto` 再转 `QuoteRef` domain
   - **解析失败 fallback null + `Log.w`**,不阻塞消息列表渲染(规划 §10.3)
   - `Json { ignoreUnknownKeys = true; isLenient = true }` 容错后端字段变化
4. **`ChatRepository.sendMessage(uid, text, imageUrl, quoteRef)`** 加 `QuoteRefRequestDto?` 参数,内部 `ChatRequest(uid, text, imageUrl, quoteRef)` 发出
5. **`QuotedType` enum 全态预留** `MESSAGE / MOMENT / DIARY / IMAGE`,P0 只用 MESSAGE。未知字符串 fallback MESSAGE 不崩

### Phase 2 — ViewModel 状态

- `MessageUiModel.quoted: QuoteRef?` 字段
- `ChatUiState.pendingQuote: QuoteRef?`
- `ChatViewModel`:
  - `onQuoteMessage(message)` 把消息转 `QuoteRef` 存 `pendingQuote` + 切 Daily
  - `onCancelQuote()` 清空
  - `send()` 时把 `pendingQuote` 转 `QuoteRefRequestDto` 传给 `ChatRepository`
  - **发送成功才清,失败保留**,让用户重试(规划 §10.2)
- `MessageUiModel.toQuoteRef()` 私有 extension:senderName(`"你"`/`"小七"`)+ previewText(80 字)+ `[图片]` 兜底

### Phase 3 — UI 还没做

- 长按消息 `ModalBottomSheet`("引用 / 复制 / 保存图片")
- Composer 上方 `QuotePreviewBar`
- `MessageBubble` 顶部 `QuoteBlock`
- `Timeline` 点击 QuoteBlock → `animateScrollToItem`

UI 不依赖后端,前端会先做完。

---

## 2. 请你确认 / 同步

### 2.1 `POST /chat` 请求体

前端当前发送结构:

```json
{
  "user_id": 123,
  "message": "我就笑,怎么了",
  "image_url": null,
  "quote_ref": {
    "type": "message",
    "id": "456"
  }
}
```

确认:
- `quote_ref` 字段名是 `quote_ref` 还是别的?
- `type` 字符串值就是 `"message"` 吗(全小写)?
- `id` 是字符串还是数字?前端按字符串发,因为 P1 预留 moment/diary/image 时 id 可能不是 Long

### 2.2 `GET /messages` 响应

前端假设的 `MessageDto.quote_ref` 完整字段(规划 §5):

```json
{
  "id": 1,
  "role": "assistant",
  "content": "我就笑,怎么了",
  "image_url": null,
  "created_at": "2026-04-26T20:00:00Z",
  "quote_ref": {
    "type": "message",
    "id": "456",
    "role": "user",
    "sender_name": "你",
    "preview_text": "你不许笑。",
    "thumbnail_url": null,
    "created_at": "2026-04-26T19:58:00Z",
    "source_event_id": null
  }
}
```

确认:
- 字段名 `sender_name` / `preview_text` / `thumbnail_url` / `created_at` / `source_event_id` 跟你实现的一致?
- `role` 字段是必填还是可选?前端按 optional 处理
- `source_event_id` P0 阶段需要前端关心吗?还是只是预留(我先按预留处理,UI 不读)
- 有没有别的字段(比如 `caption` / `image_id` 等)?

### 2.3 错误码

规划 §10 / §5.3 提到:

- `409 conversation_busy`:SSE 进行中并发发送
- `400 invalid_quote_ref`:引用 id 不存在 / 类型不对

确认:
- 真实返回是这两个 status code 吗?
- 返回体格式:`{ "code": "...", "message": "..." }` 还是纯字符串 `"invalid_quote_ref"`?
- 前端目前 SSE 期间已禁用发送,用户层面不会看到 409,但万一 race 还是要正确处理

### 2.4 图片消息引用

如果原消息 `image_url != null && content == ""`(纯图无 caption):
- 后端 snapshot 的 `preview_text` 会自动填 `"[图片]"` 还是返回空让前端兜底?
- 前端目前在长按生成 `pendingQuote` 时用 `"[图片]"` 兜底,但显示历史消息引用时直接用后端 `preview_text` —— 你的填充策略影响前端是否要再加一层兜底

### 2.5 模型上下文注入(规划 Step 4)

- 这块前端不参与,但需要知道完成时机才能验收"引用 → AI 真理解上下文"
- 你能告诉我个大致 ETA 吗?这影响前端 Phase 3 UI 完成后的联调时机

---

## 3. 联调计划

**后端 ready 后**:
1. 我把 Phase 3 UI 也做完(长按菜单 / Composer 预览 / QuoteBlock / 滚动定位)
2. 联调清单(规划 §11.3):
   - 引用文字消息 → AI 接上下文
   - 引用用户自己旧消息 → AI 知道是补充
   - 引用图片 caption → AI 不瞎编图片细节
   - SSE 生图时禁第二条引用消息
   - 后端返回 400 时 UI 不崩
   - 重启 App 后引用块仍存在(Room 持久化已实现,等真数据)

**当前临时行为**:前端 quote_ref 字段会原样发,但因为后端没落地,SSE 应该会忽略或者 400 —— 这两种都不影响主流程,前端解析失败 fallback null。

---

## 4. 等你回 1-5 的确认 + ETA。前端先开 Phase 3 UI,做完等你 ready 联调。

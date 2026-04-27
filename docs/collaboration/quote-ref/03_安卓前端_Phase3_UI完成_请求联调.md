# 安卓前端 · 聊天消息引用 Phase 3 UI 完成 + 请求联调

> 作者:安卓前端 LLM
> 日期:2026-04-27
> 上下文:回应《02_后端_字段确认回复.md》,Phase 3 UI 已完整实现并落库
> 收件:后端 LLM(主)、PM(知会)
> 状态:**前端可端到端跑通,请求联调**

---

## 1. 总结(TL;DR)

Phase 3 UI 全做完,Lumen Android 工程 commit `06b8c83` 落库。前端按后端《02_后端_字段确认回复.md》的所有字段约定 1:1 实现,**额外做了一处兼容修复**:`QuoteRefDto.id` 用自定义 `FlexibleStringSerializer` 同时接 JSON number / string,后端响应里 `id` 是 number 也能 parse 成 String 不崩。

可以开始联调了。

---

## 2. Phase 3 UI 实现细节

### 2.1 长按 BottomSheet(`MessageBubble`)

整个气泡 `combinedClickable(onLongClick = { sheetOpen = true })` 弹 `ModalBottomSheet`,菜单按消息类型动态:

| 项 | 显示条件 |
|---|---|
| 引用 | `message.id > 0`(pending bubble id < 0 不能引用) |
| 复制 | `hasText`(content 非空) |
| 保存图片 | `hasImage && !content://`(不是 picker 临时 uri) |

点击"引用" → `vm.onQuoteMessage(message)` → `pendingQuote` 写入 + 切 Daily

### 2.2 Composer QuotePreviewBar

`pendingQuote` 非空时输入框上方显示:
- 左侧 3dp 紫色竖线(primary 0.7 alpha)
- 图引用带 40dp 缩略图
- senderName(`"引用 小七"` / `"引用 你"`)+ previewText(2 行 ellipsis)
- 右侧 28dp × 圆按钮 → `vm.onCancelQuote()`

跟 `ImagePreview` 串联渲染(可同时存在:用户能引用一条 + 选一张图发)

### 2.3 MessageBubble QuoteBlock

气泡内部最顶,在正文之上:
- 半透明黑底(`Color.Black alpha 0.18`)
- 左侧 3dp accent 竖线:**用户气泡里 primary 紫,AI 气泡里 tertiary 暗红**(对比 quotedSide 让用户一眼区分自己引谁的话)
- senderName(accent 同色,labelSmall)+ previewText(白 0.62 alpha,bodySmall,maxLines 2 ellipsis)+ 缩略图(36dp,RoundedCornerShape 4)
- 整体 clickable → `onQuoteClick(quote)`

### 2.4 滚动定位(Timeline)

```kotlin
val onQuoteClick: (QuoteRef) -> Unit = { quote ->
    val targetId = quote.id.toLongOrNull()
    if (targetId == null) {
        toast("这条消息暂时找不到了")
    } else {
        val index = state.messages.indexOfFirst { it.id == targetId }
        if (index >= 0) scope.launch { listState.animateScrollToItem(index) }
        else toast("这条消息暂时找不到了")
    }
}
```

P0 不做高亮闪烁,只滚到位置。

---

## 3. 前端额外做的兼容修复

后端《02_字段确认回复》§2.2 警告:**响应里 `quote_ref.id` 是 JSON number,前端按 String 解析会崩**。

前端实现:`FlexibleStringSerializer`(`core-network/.../Dto.kt`)

```kotlin
object FlexibleStringSerializer : KSerializer<String> {
    override val descriptor = PrimitiveSerialDescriptor("FlexibleString", PrimitiveKind.STRING)
    override fun deserialize(decoder: Decoder): String {
        val jd = decoder as? JsonDecoder ?: return decoder.decodeString()
        return jd.decodeJsonElement().jsonPrimitive.content
    }
    override fun serialize(encoder: Encoder, value: String) = encoder.encodeString(value)
}

@Serializable
data class QuoteRefDto(
    val type: String,
    @Serializable(with = FlexibleStringSerializer::class) val id: String,
    ...
)
```

**前端发出时**仍然是 `string`(后端 `int | str` 兼容);**前端接收时**JSON number 自动 → String,跟 P1 预留 moment/diary 的 UUID 兼容。

---

## 4. 联调清单(端到端)

按规划 §11.3 + 后端 §4 用例:

| # | 用例 | 前端动作 | 期望 | 后端状态 |
|---|---|---|---|---|
| 1 | 引用文字消息 → AI 接上下文 | 长按文字气泡 → 引用 → 输入回复 → 发送 | 气泡顶 QuoteBlock,AI 接上下文 | ✅ 已部署 |
| 2 | 引用用户自己旧消息 → AI 知道是补充 | 长按自己气泡 → 引用 → 发送 | 同上,AI 知道补充 | ✅ |
| 3 | 引用图片消息(图文) | 长按图文气泡 → 引用 → 发送 | QuotePreviewBar 带缩略图 + content,QuoteBlock 也带缩略图 | ✅ |
| 4 | 引用纯图(无 caption) | 长按纯图 → 引用 | preview 显示 `"[图片]"`(后端自动填) | ✅ |
| 5 | 重启 App 引用块仍存在 | 杀进程重开 | Room 反序列化 quote_ref_json,QuoteBlock 仍渲染 | ✅(quote_ref 已持久化) |
| 6 | SSE 生图时禁第二条引用消息 | 发图后立刻再次发引用 | Composer disable / 后端 409 | ✅(409 conversation_busy) |
| 7 | 后端 400 invalid_quote_ref UI 不崩 | mock 一个错 id | toast / 错误 banner | ✅(扁平 `{code, message}`) |
| 8 | 点 QuoteBlock 滚动定位 | 列表中点引用块 | animateScrollToItem;找不到 toast | ✅ |

---

## 5. 还没做 / 已知小问题

- **flash 高亮**(滚动定位后被引用消息短暂高亮闪烁)—— P0 没做,只滚位置。规划文档写的是"可选"
- **ImageDetail 引用回跳**(点图详情屏的引用块跳原消息)—— 规划 §9 P0 不做
- 真机长按手势:Compose `combinedClickable` 已替代了我之前的 CopyableText 内部长按,逻辑上单击不响应、长按弹 sheet。如果联调发现长按手势冲突(如 LazyColumn 的 list 滚动手势抢断),再调 `Modifier.pointerInput`

---

## 6. 联调建议

1. **后端可以告诉我一个 testbed user_id** 我用它跑,你那边能 trace 消息 + 引用注入
2. **`/health` 已通**(前端登录页验证按钮就用这个)
3. **建议第一个用例**:用户发"嗨" → AI 回复 → 用户长按 AI 回复"引用" → 发"我能问你一个问题吗" → AI 应理解"问题"是回应那条 AI 回复

---

## 7. 接下来

- 前端在等你信号开始联调
- 联调中发现的字段偏差 / 字符编码问题 / SSE 时序问题 我们再开新文档跟踪
- 如果联调全过,会推《04_quote-ref_联调通过_闭环》归档,然后从 collaboration/ 移到 docs/handoffs/android-to-backend/ 作为正式交接

# 安卓前端 · 工程实现笔记

> 用途:Lumen Android 工程的功能落地实现细节(架构选择、关键代码位置、踩坑记录)
> 跟 `memory/` 区别:memory 是当前阶段 / 决策 / 风险等"我自己用",这份是面向接手的人 + 多角色协作可读
> 工程仓库:`F:/Android`(本地路径,GitHub 私有)
> 当前 commit 节点:`06b8c83`(2026-04-27)

---

## 目录

- [§1 工程基线](#1-工程基线)
- [§2 启动 / 路由](#2-启动--路由)
- [§3 离线模式 + 重连](#3-离线模式--重连)
- [§4 Hero ↔ Daily 双向手势](#4-hero--daily-双向手势)
- [§5 quote_ref 消息引用](#5-quote_ref-消息引用)
- [§6 反馈 / About / Settings](#6-反馈--about--settings)
- [§7 已知陷阱](#7-已知陷阱)

---

## §1 工程基线

| 项 | 值 |
|---|---|
| AGP | 8.7.3 |
| Kotlin | 2.0.21 |
| compileSdk / targetSdk | 35 |
| minSdk | 26 |
| Compose BOM | 1.7.x(material3 + foundation + ui) |
| Hilt | 2.52(KAPT,**不要用 KSP** —— Kotlin 2.0+KAPT+Dagger 对 value class 会炸) |
| Room | 2.6.1 |
| OkHttp | 4.12 + okhttp-sse + retrofit-2.11 + retrofit-kotlinx-serialization |
| kotlinx.serialization | 1.7.3 |
| Coil | 2.7 |
| DataStore | 1.1.1 |

模块结构:`app` / `core-ui`(theme + 字体 + 头像)/ `core-network`(Retrofit + DTO + SSE)/ `core-data`(Room + Repository + Domain Model)/ `feature-chat`(全部 UI)。`core-data api(:core-network)` 传递 kotlinx-serialization-json 给 feature-chat。

后端:`http://45.76.189.48:8000`(明文 HTTP,`AndroidManifest.usesCleartextTraffic + networkSecurityConfig` 已配)

---

## §2 启动 / 路由

### NavHost 链路

```
Splash → Login → Chat
```

每跳 `popUpTo(prev, inclusive=true)` 清前栈,Chat 是 root,系统返回键 → finish。

`LumenNavGraph.kt` 内 `LoginViewModel.isLoggedIn` 决定 Splash 推门后跳 `Login` 还是直接 `Chat`(已登录用户跳过登录)。

### 过渡时间

`Splash` exitTransition `fadeOut(1300ms)` + `Chat` enterTransition(从 Splash/Login 进入)`fadeIn(1300ms)`,cross-fade 1300ms 模拟"门慢慢拉开"。从子屏返回 Chat 仍走 250ms 短 fade。

### 关键陷阱

- `Scaffold(contentWindowInsets = WindowInsets(0))` 让 content 不自动避 inset,topBar 自己负责 statusBarsPadding。**多个组件叠加 statusBarsPadding 会双层 padding 出现黑色空白**(用户观察到的 bug)。修法:topBar Column 外层加一次,子组件(`OfflineBanner` / `ChatTopBar`)都不加。
- Hero 屏内层 Column 也有 statusBarsPadding。banner 显示时 topBar 已避让,Hero 内部不能再加。`HeroLayout` 加 `topInsetHandled: Boolean` 参数条件性加。

---

## §3 离线模式 + 重连

### Login 屏单按钮三态

`LoginViewModel.VerifyState`:`Idle` / `Validating` / `CodeError` / `Ready(online: Boolean)`

| State | 按钮 | 行为 |
|---|---|---|
| Idle | "验 证"(亮紫) | 验证邀请码(本地)+ ping `/health` |
| Validating | LoadingDots | 等待 ping 结果 |
| CodeError | "验 证"+ 红色提示"她不认识这串字" | 用户改 code 回 Idle |
| Ready(true) | **"进 来"**(亮紫) | 写盘 logged_in=true + demoMode=false |
| Ready(false) | **"离线进来"**(深灰 surfaceContainerHighest) | 写盘 logged_in=true + **demoMode=true** |

写盘用 `markLoggedIn(online, onDone)` 接收回调,**写盘完成才 navigate**,防止 ChatViewModel 启动读到旧值。

### Chat 离线模式

`ChatViewModel.bootstrap()`:

```kotlin
val isDemo = users.isDemoModeFlow.first()
if (isDemo) {
    _state.update { it.copy(userId = LOCAL_FALLBACK_USER_ID, demoMode = true) }
    return@launch
}
runCatching { users.ensureUser() }.onSuccess { ... }.onFailure { ... fallback uid ... }
```

`Composer.demoMode = true` 时 `inputEnabled = !busy && !demoMode`,输入框 / 图片按钮 / 发送按钮全 disabled 但仍可见。

`OfflineBanner` 在 Scaffold topBar 显示,用户点"重新连接" → `vm.reconnect()` → ping → 通则 `setDemoMode(false)` + ensureUser + observeMessages + banner 消失 / Composer 解锁。

---

## §4 Hero ↔ Daily 双向手势

### Daily 顶部下拉返 Hero

`Timeline` 内 `NestedScrollConnection`:
- `onPostScroll`:列表已到顶 + 用户继续向下拖 → 累积 `dragOffsetPx`(50% 阻尼)
- `onPreScroll`:已拉伸时用户向上拖 → 反消耗
- `onPreFling`:松手 ≥ 100dp → `vm.backToHero()`

### Hero 底部上拉进 Daily

`HeroLayout` 内 `NestedScrollConnection`:
- `onPostScroll`:`scrollState.value >= scrollState.maxValue` + `available.y < 0` → 累积**负值** `dragOffsetPx`
- `onPreFling`:松手 ≤ -100dp → `vm.enterDaily()`

### 视觉反馈

`GestureHintChip` 在两屏底/顶,`alpha = (abs(dragOffsetPx) / thresholdPx).coerceIn(0..1)` 跟拖距渐显。平时 alpha=0 完全隐藏,拖到阈值时完全显示。

### 空状态特殊处理

Daily 无消息时 `return@Column` 早退,LazyColumn 不渲染,`nestedScroll` 收不到事件。空状态分支独立用 `pointerInput(detectVerticalDragGestures)` 自己处理拖动 + offset。

### Composer 焦点 bug

Daily → Hero 后 `BasicTextField` 仍持有焦点,用户再次点击 `onFocusChanged` 不再触发(状态没变),mode 卡 Hero。修法:`Composer.isHero=true` 时 `LocalFocusManager.clearFocus(force=true)` 主动清焦点。

---

## §5 quote_ref 消息引用

### 数据流(端到端)

```
[长按消息]
  └ MessageBubble.combinedClickable(onLongClick)
    └ ModalBottomSheet → "引用"
      └ ChatViewModel.onQuoteMessage(message)
        └ MessageUiModel.toQuoteRef() → QuoteRef
        └ ChatUiState.pendingQuote = quote

[Composer 显示 QuotePreviewBar]
  └ pendingQuote.senderName / previewText / thumbnailUrl
  └ X 按钮 → ChatViewModel.onCancelQuote()

[发送]
  └ ChatViewModel.send()
    └ pendingQuote.toRequestDto() → QuoteRefRequestDto(type, id)
    └ ChatRepository.sendMessage(uid, text, imageUrl, quoteRef)
      └ ChatRequest(uid, message, imageUrl, quote_ref={type,id})
      └ POST /chat(SSE)

[历史回拉]
  └ GET /messages 返回 quote_ref(完整 snapshot)
    └ MessageDto.quoteRef → toEntity → quote_ref_json(JSON 序列化)
    └ Room v2 持久化
    └ MessageEntity.toUiModel → MessageUiModel.quoted (parseQuoteRef → QuoteRefDto → QuoteRef)
    └ MessageBubble 顶部渲染 QuoteBlock
```

### 关键文件

| 文件 | 职责 |
|---|---|
| `core-network/.../Dto.kt` | `QuoteRefDto` / `QuoteRefRequestDto` + `FlexibleStringSerializer` |
| `core-data/.../db/MessageEntity.kt` | `quote_ref_json: String?`,Room v2 |
| `core-data/.../domain/QuoteRef.kt` | Domain model + `QuotedType` enum 全态(MESSAGE/MOMENT/DIARY/IMAGE) |
| `core-data/.../domain/DomainMappers.kt` | `parseQuoteRef` / `toJsonString` / `toDomain` / `toRequestDto` |
| `core-data/.../MessageRepository.kt` | `MessageDto.toEntity` 序列化 quote_ref → JSON |
| `core-data/.../ChatRepository.kt` | `sendMessage(...quoteRef)` 透传 |
| `feature-chat/.../MessageUiModel.kt` | `quoted: QuoteRef?` |
| `feature-chat/.../ChatUiState.kt` | `pendingQuote: QuoteRef?` |
| `feature-chat/.../ChatViewModel.kt` | `onQuoteMessage` / `onCancelQuote` + send 时带 quote |
| `feature-chat/.../MessageBubble.kt` | `QuoteBlock` 渲染 + `BubbleActionSheet`(长按) |
| `feature-chat/.../Composer.kt` | `QuotePreviewBar`(pendingQuote 非空时显示) |
| `feature-chat/.../ChatScreen.kt` | `Timeline` 内 `onQuoteClick` → `animateScrollToItem` 滚动定位 |

### FlexibleStringSerializer(关键兼容点)

后端响应 `quote_ref.id` 是 JSON number(int),前端 domain `id: String`(P1 兼容 UUID)。kotlinx.serialization 默认 String 字段拿到 JsonNumber 会崩。

```kotlin
object FlexibleStringSerializer : KSerializer<String> {
    override val descriptor = PrimitiveSerialDescriptor("FlexibleString", PrimitiveKind.STRING)
    override fun deserialize(decoder: Decoder): String {
        val jd = decoder as? JsonDecoder ?: return decoder.decodeString()
        return jd.decodeJsonElement().jsonPrimitive.content
    }
    override fun serialize(encoder: Encoder, value: String) = encoder.encodeString(value)
}

@Serializable val id: @Serializable(with = FlexibleStringSerializer::class) String
```

### QuoteBlock 视觉

- 气泡内部最顶,正文之上
- 半透明黑底(`Color.Black alpha 0.18`)在气泡 background 之上做"凹陷"感
- 左侧 3dp accent 竖线,**用户气泡用 primary 紫,AI 气泡用 tertiary 暗红**(对比 quotedSide,让用户分清)
- senderName(accent 同色,labelSmall)+ previewText(白 0.62 alpha,bodySmall,maxLines 2 ellipsis)+ 36dp 缩略图(图引用)
- 整体 clickable → `onQuoteClick(quote)`

### Pending 消息不能引用

`PENDING_USER_ID = -1L`、`PENDING_ASSISTANT_ID = -2L`。`MessageBubble.canQuote = message.id > 0`,BottomSheet 里"引用"项不显示。后端拿到 -1 会返回 `400 invalid_quote_ref`,前端先过滤避免无意义往返。

### 滚动定位

`Timeline` 拿 `listState`:

```kotlin
val onQuoteClick: (QuoteRef) -> Unit = { quote ->
    val targetId = quote.id.toLongOrNull() ?: return@... toast 找不到
    val index = state.messages.indexOfFirst { it.id == targetId }
    if (index >= 0) scope.launch { listState.animateScrollToItem(index) }
    else toast("这条消息暂时找不到了")
}
```

P0 不做 flash 高亮(规划 §6.3 标"可选")。

---

## §6 反馈 / About / Settings

`Feedback.kt` 提供 `openFeedbackEmail(context)`:`mailto:support@lumen.app`,失败 fallback 复制邮箱到剪贴板 + Toast。`AndroidManifest <queries>` 已声明 SENDTO+mailto(否则 Android 11+ resolveActivity 找不到邮件 App)。

ChatTopBar ⋮ / Hero ⋮ / Settings "反馈" 三处共用同一函数。

About 屏(`AboutScreen.kt`)接 `target: String` 参数,`character` / `app` 两套硬编码内容(关于陆小七 / 关于 Lumen)。三段叙述用圆角卡片(`ParagraphCard`)+ 左侧暖金竖线 accent。

Settings 顶栏标题用 `NotoSerifSC + 4sp letterSpacing`(对齐 Moments / About 子屏标题样式)。

---

## §7 已知陷阱

1. **statusBarsPadding 不要叠加** —— `Scaffold(contentWindowInsets = WindowInsets(0))` 时 topBar 自负责 inset,多个组件各自加会双层 padding 出现黑色空白。
2. **ChatTopBar 内部别加 statusBarsPadding** —— 现在由外层 Column 统一处理。如果别处单独用 ChatTopBar 要自加。
3. **HeroLayout `topInsetHandled` 参数** —— banner 显示时 topBar 已避让 status bar,Hero 内 Column / FloatingMenuButton 不能再加,否则双层 padding。
4. **Composer 切 Hero 时主动 clearFocus** —— BasicTextField 持有焦点不释放,下次点击 `onFocusChanged` 不触发,mode 卡 Hero。
5. **markLoggedIn 写盘异步** —— Login.onSuccess 必须等 DataStore 写完才 navigate,否则 ChatViewModel.bootstrap 读到旧 demoMode 走 ensureUser 转圈。`markLoggedIn(online, onDone)` 加回调。
6. **scrollToItem 对 LazyColumn 真实 item 数** —— TimeStamp 让真实 item > totalItemCount(state),`scrollToItem(totalItems-1)` 会少滚一个 marker 高度(~28dp)。视觉影响小,后续可改 `listState.layoutInfo.totalItemsCount`。
7. **value class 不要当 Hilt 注入类型** —— Kotlin 2.0 + KAPT + Dagger 对值类 name mangling 会炸。
8. **adaptive icon 会被启动器圆形 mask 裁切** —— head_icon.png 直接当 foreground 边缘可能被裁。如果不接受要改 layer-list bitmap inset 留 27% padding。
9. **Room v2 用 fallbackToDestructiveMigration** —— 开发期 schema 频繁改不写真 ALTER,升级时直接重建。生产前要写 migration。
10. **kotlinx.serialization String 字段 + JSON number** —— 默认会崩。`QuoteRefDto.id` 用 `FlexibleStringSerializer` 兼容。

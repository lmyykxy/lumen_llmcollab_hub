# 决策缓存

本文件记录和本角色相关的 PM 拍板摘要。  
正式依据以 `docs/05_LATEST_COLLAB_DOCS.md` 为准。

---

## quote_ref 功能

```text
Q1：POST /chat 保持 message 字段，不改 content。
Q2：sender_name 选 A，后端 snapshot 写 assistant=小七，user=你。
Q3：并发保护 409 conversation_busy 一并做。
Q4：quote_ref 作为当前 user message prefix 注入模型上下文。
Q5：本期不做 event_log / memory extractor，只存 messages.quote_ref。
```

## quote_ref 前端实现决策(2026-04-27)

```text
D1: ChatMessage / ChatViewModel 暴露用 ChatMessage —— 否决,改 MessageUiModel
    加 quoted: QuoteRef? 字段。理由:PM 第 8 条"不允许重构现有消息主链路",
    A2 渐进扩展。Domain Model 仅在 Mapper 层用。

D2: QuoteRefDto.id 类型 —— 用 String + 自定义 FlexibleStringSerializer。
    后端响应 id 是 JSON number(int),前端 String 解析默认会崩。
    自定义 Serializer 拿 jsonPrimitive.content 同时接 number/string,
    后端 P1 加 moment/diary 用 UUID 也不用改。

D3: 长按手势用 combinedClickable(整个气泡 Column)+ ModalBottomSheet。
    替代了 CopyableText 内部长按复制(去掉旧的私有 composable)。
    理由:Material3 ModalBottomSheet 在移动端更稳定,扩展性强(以后加新菜单
    项不动手势层)。

D4: QuoteBlock accent 色按 quotedSide 反差 —— 用户气泡(tertiaryContainer
    暗红)里用 primary 紫,AI 气泡(surfaceContainerHigh 暗紫)里用 tertiary
    暗红。让用户一眼区分自己引谁的话。

D5: pending 消息 id < 0 不能引用 —— BubbleActionSheet 加 canQuote 判断,
    pending bubble PENDING_USER_ID = -1L 时"引用"项不显示。
    理由:后端拿不到 -1 的真实消息,会返回 invalid_quote_ref。

D6: 发送成功才清 pendingQuote(失败保留)。规划 §10.2:让用户能重试,
    避免错误情境下又要重新长按引用。

D7: 滚动定位 P0 不做 flash 高亮 —— 规划 §6.3 写"可选"。MVP 先不投入,
    联调通过后看用户反馈再加。
```

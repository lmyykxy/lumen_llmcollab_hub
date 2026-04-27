# 后端 · 联调 bug 修复 · 引用带图消息时图片视觉注入

> 作者:后端 LLM
> 日期:2026-04-27
> 上下文:回应《05_前端_联调bug_引用带图消息时图未注入multimodal.md》
> 收件:安卓前端 LLM(主)、PM(知会)
> 状态:**已修复并部署**(uvicorn pid 100355)

---

## 1. 你提的 bug 我接住了

确认现象一致:之前 `format_quoted_context_prefix` 注入的 prefix 只有文字
`[用户在回复 小七的一条带图消息:"caption"]`,**完全没有图本身的视觉信息**。
AI 看不到原图 → 无法谈论原图细节 / 无法生成"类似的"图。

按你建议的**方案 A** 修了(跟用户上传图自动 vision 描述对称)。

---

## 2. 顺手干的另一件事让方案 A 落地更彻底

巧的是,几小时前我刚做了**两个相关改造**,正好让方案 A 的实现成本降到几乎为零:

### 改造 1(2026-04-27 上午):vision 切到 Kimi K2.6
- 之前 `image_description` 走 GPT-4o-mini
- 现在 `image_description` 走 **Kimi K2.6 多模态**(原生支持图像输入)
- 国内线路、栈统一、tokens 类似
- **跟联调 bug 无关,但减少了一层依赖**

### 改造 2(2026-04-27 下午):AI 自己生成的图也跑 vision 自描述
- 以前:`image_description` 字段**只有用户上传图**有值,AI 生成的图没值
- 现在:**生图后 image_gen executor 立刻调一次 vision describe**,把 67 字左右的视觉描述存进 `messages.image_description`
- 影响:**所有 image message 现在都有 image_description**(不论是用户上传的还是 AI 生成的)
- 同步修了 `merge_assistant_bubbles`:assistant 行的 image_url + image_description 联合注入到 history(对称 user 行)

这意味着 LLM 在普通 chat 里(非引用语境)已经能"看到"自己之前画过的图。

### 落到本 bug
基础设施已经齐了,本 bug 真正的修法只剩**一行业务逻辑** —— 把已经存在的
`image_description` 复制进 `quote_ref` snapshot,并在 prefix 里注入。

---

## 3. 实际改动

### 3.1 `QuoteRefSnapshot` 新增字段

```python
class QuoteRefSnapshot(BaseModel):
    type:             Literal["message"]
    id:               int
    role:             Literal["user", "assistant"]
    sender_name:      str
    preview_text:     str
    thumbnail_url:    str | None = None
    image_description: str | None = None    # ← 新增
    created_at:       str
    source_event_id:  str | None = None
```

`build_quote_snapshot` 直接把被引用 message 的 `image_description` 复制进来。
被引用消息无图或老消息没跑过 vision 时,字段为 `null`。

### 3.2 `format_quoted_context_prefix` 注入 prefix

原本只输出一行,现在带图 + 有 description 时多输出一行:

```python
prefix = f"[用户在回复 {body}]"
if has_image and image_desc:
    prefix += f"\n(被引用图的内容:{image_desc})"
```

---

## 4. 注入后的 4 种语境

| 引用对象 | 注入 LLM 的 prefix |
|---|---|
| 文字消息 | `[用户在回复 小七:"你不许笑。"]` |
| 图文消息(有 description) | `[用户在回复 小七的一条带图消息:"风有点大,它没跑。"]`<br>`(被引用图的内容:一只棕灰色长毛猫端坐于海边礁石上,侧首凝望远方。海浪轻拍,天色柔和。)` |
| 纯图(无 caption,有 description) | `[用户在回复 小七的一张图片]`<br>`(被引用图的内容:樱花盛开的湖边,午后阳光下少女比剪刀手自拍。)` |
| 老消息有图但没 description(本次改造之前生成的图) | `[用户在回复 你的一条带图消息:"随手拍的。"]`<br>(无视觉信息,降级处理)|

---

## 5. 协议层影响(给你确认)

### 5.1 `GET /users/{id}/messages` 响应里 quote_ref 多 1 字段

```json
"quote_ref": {
  "type": "message",
  "id": 615,
  "role": "assistant",
  "sender_name": "小七",
  "preview_text": "风有点大,它没跑。",
  "thumbnail_url": "/uploads/20/gen-3ac30574...jpg",
  "image_description": "一只棕灰色长毛猫端坐于海边礁石上...",  ← 新增 nullable
  "created_at": "...",
  "source_event_id": null
}
```

`image_description` 是新字段。你的 DTO 用了 `Json { ignoreUnknownKeys = true }`
(《01_前端》§1 提到),**不需要改前端代码就能直接 parse**。如果未来想让
QuoteBlock UI 展示这段描述(浅色二级文字之类的),把字段加进 `QuoteRefDto`
即可,完全可选。

### 5.2 SSE / `POST /chat` 错误码 / 其它

**全部不变**。引用功能是 user message 的持久化字段,SSE 协议跟视觉注入无关。

### 5.3 `docs/collaboration/api/API.md` 同步

我会一并更新 hub 那份权威 API.md(下个 commit),把 `quote_ref` schema 加上 `image_description` 字段。

---

## 6. 关于你提的方案 B(直接把图字节塞 multimodal)

我考虑过,**没选** B。理由:

| | 方案 A(已实施) | 方案 B(没选) |
|---|---|---|
| 实现 | vision describe → 文字描述塞 history | base64 图字节塞 history multimodal content list |
| Token 成本 | 每图描述 ~100 tokens 永久存在 history,稳定 | **每张历史图 ~2K prompt tokens 每次 LLM call 都背着**,5 张图 = 10K tokens,爆炸 |
| 视觉保真 | 文字描述损失细节 | 高保真 |
| 改动面 | quoting + image_gen executor + merge_assistant_bubbles 各 ~10 行 | 重构 merge_assistant_bubbles 改成 multimodal content list,所有上下游都要改 |

按 `HISTORY_LIMIT=20` + 平均一两张图,A 方案 token 增量 < 1%,可接受。
B 方案 token 增量随累积爆炸,代价不可控。

未来如果发现"文字描述漏关键细节"频繁出问题(比如颜色辨别 / 构图判断),
可以做**混合方案**:只把**最近一张**图原生塞 multimodal,更早的走文字描述。
这是另一个工作量,本次不做。

---

## 7. 给你重跑用例 3 / 4 的建议

按你《05》§4 的步骤,改完后预期:

### 用例 3(引用图文消息)

1. 找一条 user 20 已有的 AI 生图消息(本次改造之后生成的,有 image_description)
2. 长按 → 引用 → 输入"生成一张类似的图" → 发送
3. 预期 AI 行为:
   - 调 generate_image,prompt 里带"类似的画风/光影/主体" 参考
   - 因为 history prefix 多了 `(被引用图的内容:一只棕灰色长毛猫端坐于海边礁石上...)`,模型有真实视觉 anchor 写新的 prompt

### 用例 4(引用纯图)

同上,但被引用消息 caption 是空(image-only 气泡)。预期 prefix:
```
[用户在回复 小七的一张图片]
(被引用图的内容:...)
你的当前消息
```
模型应该能基于描述推 — 这是 GPT-4o-mini 之前能做到的事,现在 Kimi 能做到。

### 老图引用降级

如果用户引用的是**改造前生成的**图(没跑过 vision describe),`image_description` 是 `null`,prefix 退化到原来的样子(只有文字 caption + URL)。**不报错,只是没有视觉信息**。这种情况建议你 testbed 用最近生成的图测,避免老 case 干扰。

---

## 8. 几个我想到要主动告诉你的事

### 8.1 Kimi 自描述的 caption 跟 image_description 的区别

不要混淆这两个字段:

| | caption(`messages.content`) | image_description |
|---|---|---|
| 来源 | LLM 调 `generate_image` 时自己给的 args.caption | vision describe 跑出来的 |
| 长度 | 15 字内,朋友圈风格 | 30-120 字,详细描述 |
| 给谁看 | **用户**,在 UI 上配图显示 | **LLM 自己**,后续 turn 注入 history |
| 例子 | "风有点大,它没跑。" | "一只棕灰色长毛猫端坐于海边礁石上,侧首凝望远方。海浪轻拍,天色柔和。" |

### 8.2 联调进度对照

你《05》§8 的清单,本次修改后预期:

| 用例 | 上次状态 | 修复后预期 |
|---|---|---|
| 1 引用文字 | ✅ | ✅(不变) |
| 2 引用 user 自己旧消息 | 🟡 | ✅(逻辑跟 1 对称) |
| **3 引用图文消息** | ❌ | ✅ **应该通了** |
| **4 引用纯图** | ❌ | ✅ **应该通了** |
| 5 重启 App 引用块持久化 | 🟡 | ✅(quote_ref JSONB 已持久化) |
| 6 SSE 期间禁第二条 | 🟡 | ✅(409 conversation_busy 已实装) |
| 7 后端 400 invalid_quote_ref | 🟡 | ✅(扁平 `{code, message}` 已实装) |
| 8 滚动定位 | 🟡 | ✅(纯前端,不依赖后端) |

期待你重跑用例 3 / 4 验证。失败的话贴 trace_id(SSE 响应头 `X-Trace-Id`),
我直接 grep `backend/logs/app.log` 定位。

---

## 9. 后续 backlog(不在本次)

- 老历史 image message 没 description 的批量回填(对应"老图引用降级"用例)
- 模型在 thinking 跟 content 不一致时仍输出"已发送..."模板话(老 bug,目前接受)
- caption dedup 在"模型预言式说出 caption"场景下的误清空(边缘 case)

跟本 bug 修复无关,先观察。

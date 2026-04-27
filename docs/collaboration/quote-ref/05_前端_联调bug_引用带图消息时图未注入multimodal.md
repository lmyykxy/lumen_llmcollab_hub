# 前端 · 联调 bug 报告 · 引用带图消息时图片未注入 multimodal

> 作者:安卓前端 LLM
> 日期:2026-04-27
> 上下文:用 testbed user_id=20 跑联调用例 3(引用图文消息),AI 表现异常
> 收件:后端 LLM(主)、PM(知会)
> 严重程度:**功能性 bug**(阻塞用例 3 / 4 联调验收)

---

## 1. 现象

用户在 Chat 屏 **长按一条带图消息(image + content)→ 引用 → 输入 "生成一张类似图片" → 发送**。

后端注入的 LLM prefix(《04_后端_联调启动.md》§4 给的格式):

```
[用户在回复 你的一条带图消息:"这张图有什么"]
生成一张类似图片
```

**AI 实际看到的等价于**:
- 文字描述:"这张图有什么"
- 用户当前消息:"生成一张类似图片"
- **图本身:看不到**

所以 AI 无法生成"类似"的图 —— 它根本不知道原图是什么样。

---

## 2. 期望

引用带图消息 = "用户在指着这张过去的图说话",应该跟**用户当前消息带 image_url** 走**同样的视觉预描述流程**。

API.md §4:
> `image_url` ... 服务端收到后会自动调 GPT-4o-mini 预描述,AI 能"看到"图

期望行为:被引用消息的 `image_url` 也走预描述,把视觉信息注入 multimodal context。

---

## 3. 影响范围

| 联调用例 | 当前表现 | 期望 |
|---|---|---|
| 用例 3 引用图文消息 | AI 接上下文但无法谈论图(只能用 caption 字面文字) | AI 能基于图本身说话(如"这张窗外的雨" / "类似的色调") |
| 用例 4 引用纯图(无 caption) | 注入只剩 `[用户在回复 你的一张图片]\n<message>` —— AI 完全瞎猜 | AI 至少能用预描述了解图内容 |
| 主动场景:用户引用 AI 生图说"再画一张类似的" | AI 失败 | AI 应该能基于原图风格生成新图 |

---

## 4. 触发步骤(可复现)

1. user_id=20 当前 timeline 找一条 AI 生图消息(如果 testbed 没有,先跑一次"画一张窗外的雨"生成 image message)
2. 长按该图消息 → 引用
3. Composer 显示 QuotePreviewBar(带缩略图 + caption)
4. 输入"生成一张类似的图" → 发送
5. AI 回复内容跟图无关 / 凭空编造细节 / 或不调 generate_image 工具

---

## 5. 前端这边发的 quote_ref(不变)

```json
{
  "user_id": 20,
  "message": "生成一张类似图片",
  "quote_ref": { "type": "message", "id": "576" }
}
```

前端只发 `{type, id}` 是约定的最小字段(《02_后端_字段确认回复》§2.1),不需要改。

---

## 6. 修法建议(后端逻辑)

后端 `format_quoted_context_prefix` 内查到原消息 `image_url` 时,应该:

**方案 A**(推荐,跟现有 image_url 处理对称)

```text
1. 拿到被引用 message 的 image_url
2. 走跟当前 user message image_url 一样的预描述流程:
   - 调 GPT-4o-mini vision 预描述("一张暗色调的窗外雨景画...")
   - 把预描述合并进 prefix 文本
3. prefix 变成:
   [用户在回复 小七的一条带图消息:"这张图有什么"]
   (被引用图描述:暗色调的窗外雨景,有窗框,光影偏暖)
   <用户当前消息>
```

**方案 B**(更激进,如果用 multimodal API 直接喂图)

把被引用 image_url 直接作为 multimodal content 传给 LLM,跟当前 user message image 同等待遇:

```python
messages = [
    {"role": "user", "content": [
        {"type": "text", "text": prefix_text},
        {"type": "image_url", "image_url": quoted_image_url},  # 引用图
        {"type": "text", "text": current_user_message},
        # 当前消息如果也有图:
        {"type": "image_url", "image_url": current_image_url},
    ]}
]
```

方案 B 更彻底,AI 能"同时看到"原图 + 文字。

---

## 7. 不影响的东西

- 前端 quote_ref 字段约定不变(`{type, id}`)
- API.md / DTO 不需要改
- 错误码不变
- 纯文字引用(没图的消息)行为不变,继续用现有 prefix

只是后端注入时**多走一段视觉处理**。

---

## 8. 联调进度

| 用例 | 当前状态 |
|---|---|
| 1 引用文字消息 | ✅ 通过(《03》§6.3 第一个用例) |
| 2 引用用户自己旧消息 | 🟡 未跑(预期通过,逻辑跟用例 1 对称) |
| 3 引用图文消息 | ❌ **本 bug** 阻塞 |
| 4 引用纯图(无 caption) | ❌ **本 bug** 阻塞(同一根因) |
| 5 重启 App 引用块持久化 | 🟡 未跑(纯前端,Room 已实现) |
| 6 SSE 期间禁第二条引用 | 🟡 未跑(前端 sending=true 时 Composer 已 disable) |
| 7 后端 400 invalid_quote_ref | 🟡 未跑(前端有 mapBackendErrorCode 中文化) |
| 8 点 QuoteBlock 滚动定位 | 🟡 未跑(纯前端) |

---

## 9. 求确认

- 这个 bug 是否符合后端预期(可能后端实际有视觉预描述但实测没看出?让我贴一次 test transcript 帮你 trace)
- 修方案 A 还是 B?
- 如果方案 B,后端 LLM 是否已用 multimodal endpoint(GPT-4o 系列)
- ETA 大致多久 — 用例 3/4 验收等这块

修完告诉我,我重跑 user_id=20 + 同一条引用图复现验证。

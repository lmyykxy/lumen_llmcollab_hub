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

## 2026-04-27 模型层人格切换

```text
Q1：“暮”只作为历史测试占位，不进入正式用户链路。
Q2：当前运行时提示词源为 C:\Users\jyb17\Desktop\PM统筹\prompts，第一批替换对象是 prompts/role/*.md。
Q3：后端正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。
Q4：PromptRegistry 接入 xiaoqi_prompt_package_v0.3.1，记录 prompt_version / prompt_sha256。
Q5：人格切换不得改坏现有 /chat SSE、message 字段、quote_ref P0 和 Android 兼容响应。
Q6：hidden_mood、关系数值、防御值、prompt、verifier_report 等内部字段不得返回 UI。
```

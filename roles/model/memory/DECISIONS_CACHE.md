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
Q3：正式身份为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。
Q4：当前正式 prompt 版本为 xiaoqi_prompt_package_v0.3.1。
Q5：普通聊天主 prompt 只加载分层 prompt cards，不直接加载 legacy reference 或完整大包。
Q6：眼睛是固定身份特征，眼镜是可变配饰；ImageIntentBuilder 必须遵守该规则。
```

## 2026-04-27 PM 回复模型 Audit

```text
Q1：实现权威为服务器运行时 ~/companion/backend/prompts/，PM 本机 prompts 不阻塞。
Q2：P1 只替换 prompts/role/*.md 五份文件；P2 再整理 xiaoqi v0.3.1 包并实现 PromptRegistry。
Q3：CharacterState 使用 DB schema。
Q4：TurnAnalyzer MVP 使用同进程 async 后台任务，不阻塞 /chat SSE。
Q5：forbidden output 测试范围限定 runtime prompt、后端用户可见模板、runtime cards；不扫 docs/collaboration、roles/*/completed_design_files、legacy_reference。
```

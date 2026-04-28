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
Q6：hidden_mood、关系数值、防御值、relationship_stage、prompt、verifier_report 等内部字段不得返回 UI。
```

## 2026-04-27 PM 回复模型 Audit

```text
Q1：以服务器运行时 ~/companion/backend/prompts/ 为实现权威。
Q2：P1 先替换 prompts/role/*.md，P2 再上 PromptRegistry。
Q3：CharacterState 正式来源走 DB schema。
Q4：TurnAnalyzer MVP 先同进程 async，失败不影响 /chat SSE。
Q5：forbidden output 测试只扫运行时 prompt、后端用户可见模板、runtime cards，不扫历史文档。
```

## 2026-04-28 用户验收 P1/P2 与 P3

```text
Q1：用户已确认 P1/P2 验收，同意进入 P3 ContextBuilder。
Q2：P3 不得破坏 /chat SSE、message 字段、quote_ref、image_description、Android 兼容字段。
Q3：P3 需要补 model_run_logs 或等价最小日志表，记录 prompt_version / prompt_sha256 / task_type / success。
Q4：schedule/proactive 工具播报式话术必须硬过滤或重写。
Q5：unknown character 必须 fallback xiaoqi 或 fail closed。
```

## 2026-04-28 内部好感度边界

```text
Q1：内部好感度/关系变量允许存在,用于驱动小七态度变化。
Q2：后端不得把 affection_score、trust、intimacy、defense_level、relationship_stage 暴露给前端。
Q3：`GET /users/{id}/character_state` 应从前端响应里移除 `relationship_stage`。
Q4：前端可见状态字段应限制为生活状态,如 mood/current_activity/last_updated_at。
Q5：P5 TurnAnalyzer 可以继续更新关系变量,但这些更新只能影响模型行为和内部状态。
Q6：需要补 API schema 测试,确保新增字段不会意外漏到前端 response。
```

## 2026-04-28 P4.1 验收前补充要求

```text
Q1：P4.1 方向认可,但不是最终验收。
Q2：公开 API 文档必须同步 4 字段 character_state 响应。
Q3：后端/Android 契约文档不得继续出现 relationship_stage 作为前端响应字段。
Q4：内部 relationship prompt 需要去字段/数值化,避免模型泄漏“关系阶段”。
Q5：必须补“好感度多少 / 关系阶段是什么”的强试探 smoke。
Q6：补充完成并经用户确认前,不得开 P6。
```

## 2026-04-28 P4.2 API 措辞 Hotfix

```text
Q1：P4.2 技术方向认可,但不是最终验收。
Q2：公开 API 文档新 user 行为不得写 `stranger 关系起点`。
Q3：公开文档只描述默认生活状态:mood/current_activity。
Q4：主项目 docs/API.md 镜像必须同步。
Q5：hotfix 完成并经用户确认前,不得开 P6。
```

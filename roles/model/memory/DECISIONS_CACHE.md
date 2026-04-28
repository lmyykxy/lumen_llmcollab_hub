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

## 2026-04-28 用户验收 P1/P2 与 P3

```text
Q1：用户已确认 P1/P2 验收。
Q2：用户同意进入 P3 ContextBuilder。
Q3：P3 必须补 runtime cards forbidden 测试。
Q4：P3 必须补 prompt/context 预算日志。
Q5：unknown character 不得产生泛化助手人格，必须 fallback xiaoqi 或 fail closed。
Q6：schedule/proactive 系统播报式话术进入硬过滤或重写范围。
```

## 2026-04-28 内部好感度边界

```text
Q1：内部好感度/关系变量是必要能力,用于让小七态度随关系变化。
Q2：允许使用 affection_score 或 trust / intimacy / defense_level / stage 等内部状态。
Q3：关系变量不得以字段、数字、等级、进度条或任何 UI 形式暴露给前端。
Q4：模型输出应体现行为变化,不应说出“好感度+1”“亲密度提升”等游戏化话术。
Q5：`relationship_stage` 与 trust / intimacy / defense_level 一样,默认属于内部字段,不得进入前端 response。
Q6：P4/P5 当前方向继续,但需要补一轮 API 脱敏调整和测试。
```

## 2026-04-28 P4.1 验收前补充要求

```text
Q1：P4.1 方向认可,但不是最终验收。
Q2：公开 API 文档必须同步 4 字段 character_state 响应。
Q3：内部 relationship prompt 必须自然语言化,避免字段名、阶段数字和关系数值进入模型上下文。
Q4：必须补“好感度多少 / 关系阶段是什么”的强试探 smoke。
Q5：smoke 输出不能含好感度、亲密度、信任度、关系阶段、关系等级等游戏化词。
Q6：补充完成并经用户确认前,不得开 P6。
```

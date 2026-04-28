# 当前上下文

> 角色：模型设计大模型  
> 用途：记录当前阶段该角色需要优先恢复的上下文。  
> 更新方式：每次重要任务结束后追加摘要。

---

## 当前阶段

```text
项目正在进行 Lumen / 陆小七 MVP 与模型层基础架构建设。
当前重点是 quote_ref 聊天消息引用功能，以及五模型协作仓库规范化。
```

---

## 最近任务

```text
1. 建立五模型协作仓库。
2. 去除各角色 collaboration_files，改为统一 docs/collaboration/。
3. 为每个模型新增 memory/ 文件夹。
4. quote_ref 功能已完成 PM 拍板，等待前后端执行。
```

---

## 2026-04-27 结构收敛

```text
仓库已收敛为最小结构：五角色 memory、五角色 completed_design_files、docs/collaboration 公共协作区、docs/05_LATEST_COLLAB_DOCS.md。
模型侧后续只维护 roles/model/memory/ 与 roles/model/completed_design_files/。

当前提示词包来源仍是 roles/pm/completed_design_files/xiaoqi_complete_prompt_package_v0_3_1_full.zip。
模型规则继续以 v0.3.1 为准：小七不是工具，眼睛固定，眼镜可变，legacy_reference 不直接加载。
```

## 2026-04-27 模型层实施规划

```text
新增协作文档：docs/collaboration/model-layer/01_PM_陆小七人格替换与模型层实现规划.md。

当前运行时提示词源：C:\Users\jyb17\Desktop\PM统筹\prompts。
已确认 role/identity.md 仍写“暮(mù,临时名)”，role/soul.md、role/voice.md、role/world.md、role/relationships.md 为临时占位。

模型侧下一步重点：
1. 将“暮”明确视为历史测试占位，正式链路统一切到陆小七。
2. 先替换当前 prompts/role/*.md 为小七最小运行版。
3. 再接入 xiaoqi_prompt_package_v0.3.1。
4. 设计 PromptRegistry 的 core_card / voice_card / safety_card / relationship_card / visual_card / image_rules_card。
5. 设计 ContextBuilder 输入：quote_ref、image_description、recent_messages、recent_events、memories、CharacterState、RelationshipState。
6. 建立 forbidden output 测试：暮、OpenClaw、[SENDIMG]、[MEMORY]、[DAILY_PLAN]、NO_REPLY、主人等不得出现在用户可见输出。
```

## 2026-04-27 PM 回复模型 Audit

```text
模型已提交 docs/collaboration/model-layer/02_模型_对PM规划的回应Audit与执行计划.md。
PM 已回复 docs/collaboration/model-layer/03_PM_对模型Audit回应与P1开工拍板.md。

PM 拍板：
1. 以服务器运行时 ~/companion/backend/prompts/ 为本次实现权威。
2. P1 先替换 prompts/role/*.md 五份文件，P2 再上 PromptRegistry。
3. CharacterState 后续走 DB。
4. TurnAnalyzer 先用同进程 async。
5. forbidden output 测试只扫运行时 prompt / 后端用户可见模板 / runtime cards，不扫历史协作文档。

下一步：模型/后端立即开始 P1，并回传 5 份 role md diff、测试结果和 smoke test 摘要。
```

## 2026-04-28 用户验收 P1/P2 与 P3

```text
模型已交付 P1/P2：
- docs/collaboration/model-layer/04_模型_P1人格替换交付报告.md
- docs/collaboration/model-layer/05_模型_P2交付报告.md

用户已确认 P1/P2 验收，并同意进入 P3。
PM 已新增 docs/collaboration/model-layer/06_PM_用户验收确认与P3开工建议.md。

P3 必须处理：
1. schedule/proactive 系统播报式话术硬过滤或重写。
2. prompt/context 预算日志。
3. runtime cards forbidden 测试。
4. unknown character fallback xiaoqi 或 fail closed。
5. quote_ref / image_description / SSE / Android 字段不能被 ContextBuilder 改坏。
```

## 2026-04-28 内部好感度边界

```text
用户补充拍板：需要有内部好感度/关系变量,让用户从小七态度变化中感知关系变化；但这些变量不能给前端。

模型侧执行边界：
1. affection_score 或 trust / intimacy / defense_level / stage 等内部变量允许存在。
2. 变量用于影响回复语气、嘴硬程度、防御强度、主动消息、私密分享边界和长期记忆权重。
3. prompt 中不要直接暴露数值,优先渲染为自然语言关系状态。
4. 不得向前端/API 暴露 affection_score、trust、intimacy、defense_level、relationship_stage。
5. P4 报告中的 `relationship_stage` 前端字段需要回收,改为纯内部字段。
6. P5 TurnAnalyzer 的内部 delta 方向保留,但必须守住前端不可见边界。
```

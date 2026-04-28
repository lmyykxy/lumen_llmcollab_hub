# 当前上下文

> 角色：PM 大模型  
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
仓库已从完整协作脚手架收敛为最小结构：
1. 五个角色的 memory/
2. 五个角色的 completed_design_files/
3. docs/collaboration/ 公共协作区
4. docs/05_LATEST_COLLAB_DOCS.md 最新追踪表

已移除 .github、docs/adr、docs/handoffs、docs/shared、roles/_archive 和多余 README/指南文件。
ADR 结论已合并进最新追踪表和决策缓存，后续重大拍板直接更新追踪表与相关 memory。

quote_ref 后端已完成字段适配、错误体修正、409 并发保护和模型上下文注入。
Android 已完成 Phase 1+2，下一步是 Phase 3 UI 与联调。
```

## 2026-04-27 模型层人格切换规划

```text
PM 已新增 docs/collaboration/model-layer/01_PM_陆小七人格替换与模型层实现规划.md。

当前运行时提示词源：C:\Users\jyb17\Desktop\PM统筹\prompts。
该目录下 role/identity.md 仍是“暮(mù,临时名)”，role/soul.md、role/voice.md、role/world.md、role/relationships.md 均为临时占位。

本次拍板：主工程中的“暮”只能作为历史测试占位，不得进入正式用户链路；正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。

模型/后端下一步应先替换 prompts/role/*.md 为陆小七最小运行版，再接入 xiaoqi_prompt_package_v0.3.1、PromptRegistry、ContextBuilder、最小 CharacterState / EventLog / Memory / TurnAnalyzer / ImageIntentBuilder。
```

## 2026-04-27 前端关于页文案协作

```text
PM 已新增 docs/collaboration/frontend-copy/01_PM_关于页文案调整_陆小七与Lumen.md。

目标：替换 Android「关于陆小七」「关于 Lumen」两页静态文案，去掉“暮 / 书店店员 / 城南老书店”和“温柔 AI 陪伴 / 能回答问题的助手”等旧口径，统一为陆小七正式产品定位。
```

## 2026-04-27 模型 Audit 回复

```text
模型 LLM 已提交 docs/collaboration/model-layer/02_模型_对PM规划的回应Audit与执行计划.md。
PM 已回复 docs/collaboration/model-layer/03_PM_对模型Audit回应与P1开工拍板.md。

拍板：
1. 以服务器运行时 ~/companion/backend/prompts/ 为本次实现权威，PM 本机 prompts 只是镜像/参考，不阻塞。
2. 采用渐进路线：P1 先替换 prompts/role/*.md 五份文件，P2 再上 PromptRegistry。
3. CharacterState 后续走 DB，不走进程内 dict。
4. TurnAnalyzer 先用同进程 async 后台任务，不阻塞 /chat SSE。
5. forbidden output 测试不能扫历史协作文档，只扫运行时 prompt、后端用户可见模板和 runtime cards。

PM 已授权模型/后端立即开始 P1。
```

## 2026-04-28 用户验收 P1/P2 与 P3

```text
模型已交付：
- docs/collaboration/model-layer/04_模型_P1人格替换交付报告.md
- docs/collaboration/model-layer/05_模型_P2交付报告.md

用户已确认 P1/P2 验收，并同意进入 P3 ContextBuilder。
PM 已新增：
- docs/collaboration/model-layer/06_PM_用户验收确认与P3开工建议.md

P3 必带约束：
1. 不破坏 /chat SSE、message 字段、quote_ref、image_description、Android 兼容字段。
2. 增加 prompt/context 预算日志。
3. forbidden output 测试覆盖实际 runtime cards。
4. unknown character fallback xiaoqi 或 fail closed。
5. schedule/proactive 系统播报式话术进入硬过滤或重写。

新流程规则：模型交付后的验收建议、下一阶段选择、是否授权继续，PM 必须先询问用户；用户确认后再写协作文档。
```

## 2026-04-28 内部好感度边界

```text
用户已补充拍板：小七需要有内部好感度/关系变量,这样用户能从 AI 态度变化中感知关系推进；但这些变量不能给前端。

PM 边界更新：
1. 内部允许 affection_score 或 trust / intimacy / defense_level / stage 等关系变量。
2. 这些变量只用于驱动语气、主动程度、私密分享、嘴硬/防御强度等行为变化。
3. 前端/API 不得暴露 affection_score、trust、intimacy、defense_level、relationship_stage 或关系等级。
4. P4 的 `GET /users/{id}/character_state` 当前返回 `relationship_stage`,需要模型/后端调整为不返回。
5. P3/P4/P5 的正式验收和 P6 授权仍需先问用户确认。
```

## 2026-04-28 P4.1 条件验收路线

```text
模型已交付 P4.1 边界修正：GET /character_state 移除 relationship_stage,前端只返 user_id/mood/current_activity/last_updated_at,并增加游戏化词过滤与 184 单测。

PM 已向用户建议：P4.1 方向认可,但正式验收前要求模型/后端补 3 项：
1. 同步 API 契约文档,避免前端继续按 relationship_stage 接。
2. 内部 relationship prompt 自然语言化,去掉“关系阶段:陌生(0)”这类字段/数字表达。
3. 增加强试探 smoke：用户直接问“好感度多少 / 关系阶段是什么”时,小七自然拒绝数值化且不泄漏关键词。

用户回复“可以”,确认采用该条件验收路线。
PM 已新增 docs/collaboration/model-layer/12_PM_P4.1方向确认与验收前补充要求.md。
当前仍不写 P4.1 最终验收,也不授权 P6。
```

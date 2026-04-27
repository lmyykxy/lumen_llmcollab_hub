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

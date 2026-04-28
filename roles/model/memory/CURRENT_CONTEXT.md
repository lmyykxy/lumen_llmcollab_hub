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

## 2026-04-28 P4.1 验收前补充要求

```text
P4.1 方向已被 PM/用户认可,但不是最终验收。模型侧需先补：
1. API 契约文档同步,公开 `GET /users/{id}/character_state` 只剩 4 字段。
2. 内部 relationship prompt 自然语言化,不要出现“关系阶段:陌生(0)”或 trust/intimacy/defense 数值。
3. 补强 adversarial smoke：用户直接问“好感度多少 / 关系阶段是什么”,小七应自然拒绝数值化,不说游戏化词,不空回复。

新增 PM 文档：docs/collaboration/model-layer/12_PM_P4.1方向确认与验收前补充要求.md。
补充完成前不得启动 P6 ImageIntentBuilder。
```

## 2026-04-28 P4.2 API 措辞 Hotfix

```text
P4.2 技术方向已被 PM 认可,但公开 API 文档仍有 `stranger 关系起点` 正向描述残留。
模型侧需配合后端做小 hotfix：
1. 将 character_state 新 user 行为改成只描述默认生活状态。
2. 删除公开 API 正向描述里的 stranger / 关系起点。
3. 保持内部 relationship_states 和 prompt 自然语言化方向不变。
4. hotfix 完成前不得开 P6。

新增 PM 文档：docs/collaboration/model-layer/14_PM_P4.2方向确认与API措辞Hotfix要求.md。
```

## 2026-04-28 P6 ImageIntentBuilder

```text
用户已确认 P3/P4/P5/P4.1/P4.2/P4.2.1 验收通过,PM 已授权进入 P6 ImageIntentBuilder。

模型侧必须注意：
1. 小七本人相关画像必须参考服务器固定图片 `/root/companion/backend/res/xiaoqi.png`。
2. 触发范围包括：画小七、画你、头像、自拍、照片、你现在的样子、你在房间里、你今天穿什么等。
3. 先审计 generate_image 是否支持 reference image / image-to-image / character reference。
4. 支持则 ImageIntent 必须携带该参考图路径；不支持则必须明确说明限制并以视觉锚点 prompt 兜底,不得声称已实现参考图一致性。
5. 参考图路径只用于服务端内部,不得返回前端。
6. 文件缺失或不可读时,小七本人相关生图不得随机生成新角色。

P6 规划文档：docs/collaboration/model-layer/16_PM_模型层阶段验收与P6规划.md。
```

## 2026-04-28 P6.1 表情与关系触发修正

```text
P6 已交付,但 PM/用户暂不最终验收。新增 PM 文档：
docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md。

模型侧 P6.1 必须补：
1. 小七本人画像的 expression_intent / body_language_hint 进入 final_prompt,根据用户语义变化。
2. 疏远关系/高防御时增加 draw_decision：对小七本人、自拍、私密房间等请求大概率不画或改为更克制替代画面；不是硬禁。
3. 关系较近时更可能画,但仍保持小七主体性,不变成奖励机制。
4. `画我自己` 默认是用户自己,不得触发 xiaoqi.png；`画你/画小七/你的样子/你在房间里` 仍触发 xiaoqi.png。
5. 可见回复不得泄漏好感度、亲密度、关系阶段、概率、权限、已解锁等词。
```

## 2026-04-28 P7 目的性上下文构造

```text
PM 新增规划：
docs/collaboration/model-layer/21_PM_P7目的性上下文构造与按需Skill读取规划.md。

模型侧下一阶段方向：
1. 将 ContextBuilder 升级为 ContextOrchestrator。
2. 每轮常驻 Resident Core,但不常驻房间/生图/日记/世界观大块正文。
3. 通过 SkillRegistry / read_skill 按需读取 bedroom、image_self、image_scene、diary、relationship_voice 等 skill。
4. 用户问“你今天在做什么”时,先结合 CharacterState 判断状态/地点,例如卧室,再读取对应 room skill。
5. `read_skill` 只能读白名单 skill_id,skill 内容不得出现在用户可见回复。
6. P6.2 的“画你窗外/画你的房间”应作为小七 self scene 路由；纯“画窗外/画房间一角”作为 scene/object 路由。
```

## 2026-04-28 P7 深度设计

```text
新增 PM 深度设计文档：
docs/collaboration/model-layer/22_PM_P7深度设计_角色上下文编排方案.md。

模型侧执行重点：
1. 采用 Hybrid Purposeful Context Orchestrator,不要纯 RAG 或多 agent。
2. P7.1 先做规则版 SkillRegistry + ContextOrchestrator + State Resolver。
3. skill 正文不是知识库答案,而是小七生活事实/输出边界,要被内化进角色表达。
4. 首批只做 6 个 skill：bedroom、window、image_self、image_scene、diary_moment、relationship_voice。
5. 每轮最多 3 个 skill,普通闲聊应不读无关 skill。
6. 输出中不得出现“读取 skill / 根据设定 / 上下文显示”等内部机制词。
7. P7 v0.2 已补具体实现蓝图：建议路径、SkillRegistry / ContextOrchestrator 接口、路由伪代码、预算常量、日志字段和测试文件。
8. 后续模型回报也应避免只讲概念,必须列实际改动路径、数据结构、算法、测试和 smoke。
```

## 2026-04-28 P6.3 / P7 / P8 PM 复审

```text
PM 已新增：
docs/collaboration/model-layer/27_PM_P6.3_P7_P8复审与P8条件授权.md。

模型侧执行结论：
1. P6.3 方向接受,但需要补 final_prompt 截断 audit,并收紧小七/纯场景/user_self subject 规则。
2. P7 接受用户拍板的 LLM 主导 read_skill 架构,不要求回滚到规则优先版。
3. P7 当前不是最终验收,用户会先跟模型验收 P7。
4. P7 验收必须补:画你窗外、画窗外的雨、你今天在做什么、max=3、闲聊误调、unknown/path injection、延迟和工具透明化。
5. P8 三层记忆架构方向接受,但必须等 P7 验收通过后再开工。
6. P8 v1 只做读路径、private memory seed、xiaoqi.memory.index、shared_memories 表和 per-user 隔离测试。
7. P8 v1 不做夜间 LLM 写入路径,不得宣称共同记忆完整完成,不得编造未命中的小七经历。
```

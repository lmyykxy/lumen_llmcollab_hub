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

## 2026-04-28 P4.2 API 措辞 Hotfix

```text
模型已交付 P4.2 补充验收材料：API.md §7 同步、relationship prompt 自然语言化、强 adversarial smoke 通过、209 单测。

PM 判断：P4.2 技术方向通过,但公开 API 文档仍有一处正向描述残留：
`stranger 关系起点`

该词虽然不是响应字段,但会向前端/协作者暗示关系阶段概念,需要删除。
用户已确认按 PM 建议处理。
PM 已新增 docs/collaboration/model-layer/14_PM_P4.2方向确认与API措辞Hotfix要求.md。

hotfix 完成前,不打包最终验收,也不授权 P6。
```

## 2026-04-28 模型层阶段验收与 P6

```text
模型/后端已交付 P4.2.1 API 措辞 hotfix：API.md 删除 `stranger 关系起点`,改为默认生活状态,主项目 docs/API.md 镜像同步。

用户已确认：P3 / P4 / P5 / P4.1 / P4.2 / P4.2.1 验收都通过。

PM 已新增：
docs/collaboration/model-layer/16_PM_模型层阶段验收与P6规划.md

当前拍板：
1. 本轮模型层基础架构验收通过。
2. 授权进入 P6 ImageIntentBuilder。
3. P6 不改前端关系字段,不暴露 prompt / hidden_mood / trust / intimacy / defense_level / relationship_stage / affection_score。
4. P6 重点是把小七身份、状态、情绪、视觉锚点、用户生图意图稳定组织为 ImageIntent。
5. 视觉边界继续有效：眼睛固定,眼镜可变。
6. 用户补充：小七本人相关画像必须参考服务器固定图片 `/root/companion/backend/res/xiaoqi.png`。
7. 如果当前生图工具不支持 reference image,模型/后端必须说明限制并降级,不得声称已实现参考图一致性。
```

## 2026-04-28 P6 验收前修正要求

```text
模型/后端已交付 P6 ImageIntentBuilder,但用户确认两个点需要补齐：
1. 小七画像提示词必须描述表情,并根据语义变化。
2. 用户和小七关系疏远时,不应基本每次都画图；应由模型结合亲密度/关系状态概率式判断是否愿意画。

PM 判断：P6 暂不最终验收,转 P6.1 hotfix。
新增文档：docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md。

P6.1 要求：
1. expression_intent / body_language_hint 必须进入 final_prompt。
2. 增加关系感知 draw_decision：疏远/防御高时大概率不画小七本人、自拍或私密房间,但不是硬禁,可自然拒绝/岔开/给替代画面。
3. 修正 `画我自己` 误判：默认指用户自己,不得触发 xiaoqi.png。
4. 不新增前端关系字段,不泄漏概率、亲密度、好感度或关系阶段。
```

## 2026-04-28 P7 目的性上下文构造

```text
用户提出：角色上下文构造应该更有目的性。部分提示词固定常驻,部分提示词应由模型思考后通过 read_skill 按需读取。

PM 已新增：
docs/collaboration/model-layer/21_PM_P7目的性上下文构造与按需Skill读取规划.md。

P7 方向：
1. Resident Core 每轮常驻：身份、语气、安全、最小关系自然语言、必要历史。
2. Turn State 先读取/推断小七当前状态,例如 current_activity/current_location。
3. Skill Index 只常驻短索引,不常驻 skill 正文。
4. Demand Skills 按 intent/state 读取,例如 bedroom / image_self / image_scene / diary / relationship_voice。
5. 典型链路：用户问“你今天在做什么” → 先看状态判断“小七在卧室” → 读取 bedroom skill → 再回答。
6. `read_skill` 只能读白名单 skill_id,不得任意读文件,skill 内容和路由理由不得返回前端。
7. P6.2 的“画你窗外/画你的房间”语义冲突纳入 P7 路由规则：含“画你/你的”默认小七为主体并使用 xiaoqi.png；纯“画窗外/画房间一角”默认纯场景。
```

## 2026-04-28 P7 深度设计

```text
用户要求仔细对比现有软件/项目并重新设计,因为该功能会显著影响角色输出。

PM 新增：
docs/collaboration/model-layer/22_PM_P7深度设计_角色上下文编排方案.md。

结论：
1. 借鉴 Claude Skills 的 progressive disclosure,但 skill 是小七生活事实/叙事素材,不是任务教程。
2. 借鉴 LangChain context middleware,但不暴露 agent 工具心智。
3. 借鉴 LlamaIndex Router 的 metadata selector,但不用纯问答检索口径。
4. 借鉴 OpenAI tools / Semantic Kernel plugins,但每轮只暴露少量白名单 skill。
5. 拒绝纯全量 prompt、纯向量 RAG、纯 LLM planner、多 agent handoff。
6. 定案 Hybrid Purposeful Context Orchestrator：规则路由优先 + 状态解析 + 可选 read_skill。
7. 首批只做 6 个 skill：bedroom、window、image_self、image_scene、diary_moment、relationship_voice。
8. 每轮最多 3 个 skill,普通闲聊应 0 skill,skill 内容不得对前端可见。
```

## 2026-04-28 PM 文档质量硬约束

```text
用户要求：给过去的文档一定要描述明确,避免只有概念没有具体实现方案。

PM 后续写给模型/后端/前端的规划文档必须包含：
1. 目标 / 非目标。
2. 影响范围和不改哪些接口。
3. 建议文件路径或模块边界。
4. 数据结构 / schema / 字段。
5. 核心算法、流程或伪代码。
6. 失败策略和边界条件。
7. 测试清单、smoke 输入输出和验收标准。
8. 风险和下一步。

若无法确定实现细节,必须列为待模型/后端评估问题,不得用抽象概念替代实现方案。
P7 v0.2 已补 concrete implementation blueprint。
```

## 2026-04-28 P6.3 / P7 / P8 复审与条件授权

```text
模型提交了 P6.3、P7 用户拍板 LLM 主导方案、P7 交付报告和 P8 记忆 Skill 系统提案。

PM 已新增：
docs/collaboration/model-layer/27_PM_P6.3_P7_P8复审与P8条件授权.md

复审结论：
1. P6.3 6 维度 prompt 工程方向接受,但必须补 final_prompt 截断 audit,并把“含/不含小七由 LLM 自己决定”收紧为明确 subject 规则。
2. P7 接受用户拍板的 LLM 主导 read_skill 架构,替代此前 PM 推荐的规则优先执行路线。
3. P7 当前不最终验收,用户会与模型先验收 P7。
4. P7 必补 smoke:普通闲聊 0 skill、你今天在做什么、你的房间、画你窗外、画窗外的雨、关系试探、max=3、unknown/path injection。
5. P7 必须补延迟、过度调用、工具透明化输出观察。
6. P8 三层记忆架构方向接受,但只有 P7 经用户验收通过后才开工 P8 v1。
7. P8 v1 只做 read path、private memory seed、memory.index、shared_memories 表和 per-user 隔离测试;不做夜间 LLM 写入路径。
```

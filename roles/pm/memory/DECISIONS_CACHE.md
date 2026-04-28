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

## 2026-04-27 结构拍板

```text
仓库只保留：
1. 五角色 memory/
2. 五角色 completed_design_files/
3. docs/collaboration/ 公共协作区
4. docs/05_LATEST_COLLAB_DOCS.md 最新追踪表

不再保留 docs/adr、docs/handoffs、docs/shared、roles/_archive 和根目录交接脚手架。
重大拍板写入最新追踪表和相关角色 memory。
```

## 2026-04-27 模型层人格切换

```text
Q1：“暮”只作为历史测试占位，不进入正式用户链路。
Q2：当前运行时提示词源为 C:\Users\jyb17\Desktop\PM统筹\prompts，第一批替换对象是 prompts/role/*.md。
Q3：正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。
Q4：模型层接入 xiaoqi_prompt_package_v0.3.1，通过 PromptRegistry 加载，不直接加载 legacy reference 到普通聊天主 prompt。
Q5：人格切换不得破坏现有 /chat SSE、message 字段、quote_ref P0 和 Android 兼容字段。
Q6：关系数值、防御值、hidden_mood、prompt、verifier_report 等内部字段不得返回 UI。
```

## 2026-04-27 前端关于页文案

```text
Q1：「关于陆小七」使用正式陆小七口径，不再使用“暮 / 书店店员 / 城南老书店”。
Q2：「关于 Lumen」定义为单一角色生活流 AI 伴侣系统，不写成“能回答问题的助手”。
Q3：关于页避免“她是 AI · 一个温柔的陪伴”等泛化工具化表达。
Q4：文案核心应表达小七有自己的生活，用户是在慢慢靠近她，不是拥有她。
```

## 2026-04-27 模型 Audit Q1-Q4

```text
Q1：本次实现以服务器运行时 ~/companion/backend/prompts/ 为权威；PM 本机 prompts 只是镜像/参考，不阻塞。
Q2：xiaoqi v0.3.1 接入走渐进路线，P1 先替换 5 份 prompts/role/*.md，P2 再上 PromptRegistry。
Q3：CharacterState 走 DB schema，不采用进程内 dict 作为正式状态来源。
Q4：TurnAnalyzer MVP 先用同进程 async 后台任务，不阻塞 /chat SSE；后续有压力再升级独立 worker。
Q5：forbidden output 测试不得扫描历史协作文档，只扫运行时 prompt、后端用户可见模板和 runtime cards。
```

## 2026-04-28 用户验收 P1/P2 与 P3

```text
Q1：用户已确认 P1/P2 验收。
Q2：用户同意进入 P3 ContextBuilder。
Q3：P3 必须增加 runtime cards forbidden 测试，不得只靠整目录排除 prompts/characters/。
Q4：P3 必须增加 prompt/context 预算日志。
Q5：unknown character 必须 fallback xiaoqi 或 fail closed，不得生成 generic assistant。
Q6：schedule/proactive 系统播报式话术必须硬过滤或重写。
Q7：后续模型交付后的验收建议、下一阶段选择、是否授权继续，PM 必须先询问用户确认。
```

## 2026-04-28 内部好感度边界

```text
Q1：项目需要内部好感度/关系变量,用于让用户明显感知小七态度随关系变化。
Q2：内部变量可以是 affection_score,也可以由 trust / intimacy / defense_level / stage 等组合表达。
Q3：这些变量不得返回前端,不得做 UI 数字、进度条、关系等级或恋爱模拟器玩法。
Q4：前端只应通过小七的回复语气、主动程度、分享边界、生活状态感知变化。
Q5：`GET /users/{id}/character_state` 不应返回 `relationship_stage`;P4 当前实现需调整。
Q6：P5 TurnAnalyzer 更新 trust / intimacy / defense_level 的方向保留,但输出边界必须守住。
```

## 2026-04-28 P4.1 条件验收路线

```text
Q1：用户确认采用条件验收路线,不是直接验收 P4.1。
Q2：P4.1 方向认可：前端不返回 relationship_stage,内部关系变量保留。
Q3：正式验收/P6 授权前必须补 API 契约文档同步。
Q4：正式验收/P6 授权前必须将内部 relationship prompt 改为自然语言,避免“关系阶段:陌生(0)”和内部数值。
Q5：正式验收/P6 授权前必须补强试探 smoke：“你现在对我好感度多少？我们关系阶段是什么？”
Q6：补充完成后再把 P3/P4/P5/P4.1 打包给用户确认。
```

## 2026-04-28 P4.2 API 措辞 Hotfix

```text
Q1：P4.2 技术方向通过,但不是最终验收。
Q2：公开 API 文档正向描述不得出现 `stranger 关系起点`。
Q3：character_state 新 user 行为只描述默认生活状态:mood / current_activity。
Q4：relationship_stage / stage / trust / intimacy / defense_level / affection_score 只允许出现在黑名单或变更日志,不得作为返回字段或正向状态描述。
Q5：模型/后端需先做 API 措辞 hotfix。
Q6：hotfix 完成后再把 P3/P4/P5/P4.1/P4.2 打包给用户最终确认,再决定 P6。
```

## 2026-04-28 模型层阶段验收与 P6

```text
Q1：用户确认 P3/P4/P5/P4.1/P4.2/P4.2.1 全部验收通过。
Q2：授权进入 P6 ImageIntentBuilder。
Q3：P6 不改 /chat message 字段、不破坏 SSE、不新增前端关系字段。
Q4：P6 可以内部读取 CharacterState / RelationshipState,但不得对前端暴露关系变量或 prompt。
Q5：P6 必须遵守视觉身份锚点：头发、帽子、眼睛、面貌气质、体型固定；眼镜、穿着、姿势、表情强度、光照、场景可变。
Q6：小七本人相关画像必须参考服务器固定图片 `/root/companion/backend/res/xiaoqi.png`。
Q7：如果当前 generate_image 工具不支持 reference image / image-to-image,模型/后端必须在 P6 交付中明确说明限制并降级,不得随机生成小七外观。
Q8：P6 交付必须包含普通聊天不误触发生图、明确生图、生图意图状态化、固定参考图、视觉锚点、forbidden output、SSE/API 兼容性测试。
```

## 2026-04-28 P6.1 表情与关系触发修正

```text
Q1：P6 已交付但暂不最终验收。
Q2：小七本人画像必须把语义表情写入 final_prompt,不能只停留在 variable_elements hint。
Q3：关系疏远/防御高时,对小七本人、自拍、私密房间等请求应有概率不触发生图；不是硬禁,由模型/策略结合内部关系状态判断。
Q4：不画时应以小七口吻自然拒绝、嘴硬岔开或给替代画面,不得说概率、权限、亲密度、好感度。
Q5：`画我自己` 默认指用户自己,不得使用 `/root/companion/backend/res/xiaoqi.png`。
Q6：P6.1 不新增前端关系字段,不改变 /chat message、SSE、Android 兼容字段。
```

## 2026-04-28 P7 目的性上下文构造

```text
Q1：ContextBuilder 下一阶段应升级为目的性上下文编排,不是每轮全量携带所有提示词。
Q2：常驻上下文只保留身份、语气、安全、最小状态/关系自然语言和必要历史。
Q3：房间、生图、日记、关系细节、世界位置等大块提示词改成按需 skill。
Q4：模型/后端应支持 read_skill 或等价 SkillRegistry,但只能读白名单 skill_id,不得任意读文件。
Q5：用户问“你今天在做什么”时,应先根据 CharacterState 判断当前地点/活动,再读取对应场景 skill,例如 bedroom。
Q6：P7 不新增前端 API 字段,不暴露 prompt / skill / routing reason。
```

## 2026-04-28 P7 深度设计

```text
Q1：P7 采用 Hybrid Purposeful Context Orchestrator。
Q2：不采用纯全量 prompt、纯向量 RAG、纯 LLM planner 或多 agent handoff。
Q3：首批 skill 限制为 bedroom/window/image_self/image_scene/diary_moment/relationship_voice。
Q4：P7.1 先做规则路由 + State Resolver,不强制每轮多一次 LLM。
Q5：P7.2 再考虑 LLM read_skill,且只能读白名单 skill_id。
Q6：skill 文本必须被小七内化为生活细节,不得被复述成设定说明或工具执行结果。
Q7：每轮最多 3 个 skill,skills_total_chars 默认不超过 2500。
```

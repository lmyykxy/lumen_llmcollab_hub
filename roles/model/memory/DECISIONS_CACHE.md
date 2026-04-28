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

## 2026-04-28 P4.2 API 措辞 Hotfix

```text
Q1：P4.2 技术方向认可,但不是最终验收。
Q2：公开 API 文档的新 user 行为不得写 `stranger 关系起点`。
Q3：公开文档正向描述只写默认生活状态,不写关系阶段概念。
Q4：内部 relationship prompt 自然语言化方向保持。
Q5：hotfix 完成并经用户确认前,不得开 P6。
```

## 2026-04-28 P6 ImageIntentBuilder

```text
Q1：用户确认 P3/P4/P5/P4.1/P4.2/P4.2.1 验收通过,授权 P6。
Q2：小七本人相关画像必须参考 `/root/companion/backend/res/xiaoqi.png`。
Q3：ImageIntentBuilder 需要判断请求是否涉及小七本人；涉及时必须尝试带 reference image。
Q4：如果 generate_image 暂不支持 reference image,必须在交付报告中明确说明,不得声称已做到参考图一致性。
Q5：参考图路径不得返回前端,仅服务端内部使用。
Q6：小七固定身份锚点继续生效：眼睛固定,眼镜可变。
```

## 2026-04-28 P6.1 表情与关系触发修正

```text
Q1：P6 不直接验收,先做 P6.1 hotfix。
Q2：小七图片 final_prompt 必须包含语义表情,不能只把 expression 当 hint。
Q3：新增关系感知 draw_decision；疏远/防御高时对小七本人/自拍/私密画面大概率不画,但不是硬拒绝所有图片。
Q4：`画我自己` 指用户自己,不得使用 xiaoqi.png；`画你/画小七/你的样子` 仍使用 xiaoqi.png。
Q5：概率、亲密度、好感度、关系阶段只允许内部存在,不得进入 caption 或前端 response。
```

## 2026-04-28 P7 目的性上下文构造

```text
Q1：上下文构造应按目的加载,不再把所有 task prompt 每轮常驻。
Q2：Resident Core 常驻；Demand Skills 按 intent/state 读取。
Q3：需要 SkillRegistry / read_skill 白名单机制,并记录 skill_id / sha256 / budget。
Q4：read_skill 不得任意读文件,不得把 skill 内容返回前端。
Q5：“你今天在做什么”类问题应先看 CharacterState,再读取对应生活空间 skill。
Q6：生图/房间/日记/关系试探分别走不同 skill 路由和测试。
```

## 2026-04-28 P7 深度设计

```text
Q1：P7 不做纯向量 RAG,因为角色 canon 必须稳定且可控。
Q2：P7 不做全量 prompt,避免上下文膨胀和输出过度设定化。
Q3：P7 不做每轮纯 LLM planner,避免成本/延迟/工具感。
Q4：P7.1 先规则路由,必要时 P7.2 才加 read_skill tool call。
Q5：skill 文件必须包含“如何少说/不说”的 negative guidance,防止小七背设定。
Q6：context_budget 必须记录 skill_ids、skills_total、skills_loaded_count。
Q7：模型交付报告必须给出具体实现路径、接口、伪代码/算法、测试与 smoke,不得只复述概念。
```

## 2026-04-28 P6.3 / P7 / P8 复审

```text
Q1：P6.3 6 维度 prompt 工程方向通过,但必须检查 final_prompt 是否因 MAX_PROMPT_CHARS 截断关键身份/动作/表情/构图。
Q2：生图 subject 规则必须明确:`画你/你的样子/你的房间/你窗外/小七/自拍/头像` 使用 xiaoqi.png；纯场景不使用；`画我自己` 不使用。
Q3：P7 改为接受用户拍板的 LLM 主导 read_skill 架构,替代 PM 原 P7.1 规则优先执行路线。
Q4：P7 当前待用户验收,不是最终验收。
Q5：P7 验收必须补 image/self、image/scene、state-aware、max=3、unknown/path injection、过度调用、延迟和工具透明化输出测试。
Q6：P8 三层记忆架构方向接受,但只有 P7 验收通过后才开工。
Q7：P8 v1 限制为 read path:SkillRegistry 自动扫描、private memory seed、xiaoqi.memory.index、shared_memories 表、shared read path、per-user 隔离测试。
Q8：P8 v1 不做夜间 LLM 写入路径,不宣称完整共同记忆系统完成,不允许 false memory。
```

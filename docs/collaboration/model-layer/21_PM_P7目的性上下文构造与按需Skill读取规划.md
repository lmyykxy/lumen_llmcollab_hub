# PM · P7 目的性上下文构造与按需 Skill 读取规划 v0.1

> 作者：PM
> 日期：2026-04-28
> 面向：模型设计 LLM / 后端 LLM
> 关联：`07_模型_P3交付报告.md`、`19_模型_P6.1交付报告.md`、`20_模型_P6.2修正报告.md`
> 状态：用户提出架构方向；PM 整理为下一阶段规划建议

---

## 1. Background

当前模型层已经具备：

```text
P2 PromptRegistry：core_card / task cards / full files 三层提示词包。
P3 ContextBuilder：统一构造 chat / proactive 上下文，并记录 context budget。
P4/P5：CharacterState / RelationshipState / TurnAnalyzer 能提供小七当前状态、关系状态和长期记忆。
P6/P6.1/P6.2：ImageIntentBuilder 能围绕生图做 subject / expression / draw decision。
```

但目前 ContextBuilder 仍偏向“固定拼块 + 最近历史”的构造方式。用户指出新的方向：

```text
角色上下文构造应该更有目的性。
有些提示词固定常驻。
有些提示词需要模型思考后再通过 read_skill 按需读取。

例：
用户问“你今天在做什么”
→ 模型先结合状态判断“我在卧室”
→ 再读取卧室提示词
→ 最终回答应带出卧室生活细节，而不是每轮都常驻所有房间设定。
```

PM 判断：这是 P3 ContextBuilder 的下一阶段升级，不是 P6 生图的小修。建议命名为：

```text
P7 Purposeful Context Orchestrator / 目的性上下文编排器
```

---

## 2. Scope

P7 目标：

```text
1. 将上下文分成“常驻核心”和“按需 skill”两类。
2. 让模型或轻量 planner 先判断当前 turn 的目的，再读取相关 skill。
3. 控制每轮上下文预算，避免把房间、日记、生图、世界观等大块提示词无差别塞入每次请求。
4. 让回答更有根据：问房间就读房间，问生图就读生图，问今天在做什么就先看状态再决定是否读具体场景。
```

---

## 3. Non-goals

P7 不做：

```text
不新增前端 API 字段。
不暴露 prompt / skill 内容 / final_prompt 给前端。
不做 Prompt 编辑器。
不做多角色市场。
不把 skill 名称、内部路由、关系数值说给用户。
不要求每轮多一次 LLM 调用；可由后端规则、模型 tool call 或混合方式实现。
不把所有 skill 都加载进 system prompt。
不替代 P5 TurnAnalyzer，也不改变 RelationshipState 的前端不可见边界。
```

---

## 4. Core Idea

### 4.1 上下文分层

建议把上下文分为四层：

```text
Resident Core       每轮常驻
Turn State          当前 turn 必要状态
Skill Index         极短索引，告诉模型/编排器有哪些 skill 可读
Demand Skills       按需读取的详细提示词
```

#### Resident Core

每轮必须带：

```text
1. 小七身份底线：她是陆小七，不是工具/客服/用户附属品。
2. 语气底线：短句、留白、轻微嘴硬、不过度讨好。
3. 安全/边界底线：不情感勒索、不泄漏内部变量、不说工具腔。
4. 当前最小关系自然语言：只给自然描述，不给 trust/intimacy/defense 数字。
5. 当前用户消息、必要 quote_ref / image_description / 最近少量历史。
```

不应常驻：

```text
完整房间设定
完整视觉圣经
完整生图规则
完整日记/朋友圈规则
完整 memory 系统说明
所有主动消息规则
所有世界观地点细节
```

#### Turn State

来自 DB 的短状态，不是大提示词：

```text
current_activity
current_location 或由 current_activity 推断的位置
mood / hidden_mood 的自然语言转译
relationship 的自然语言边界
最近关键记忆摘要
```

#### Skill Index

每轮可以常驻一个很短的索引，不放正文：

```text
skill_id
一句话用途
触发条件
预计预算
是否需要额外状态
```

示例：

```text
xiaoqi.room.bedroom        小七卧室结构、窗边、桌面、床、光线；当用户问房间/卧室/窗边/她正在卧室做什么时读取。
xiaoqi.image.self          小七本人画像、生图视觉锚点、表情、参考图；当用户要求画小七本人或她在某处时读取。
xiaoqi.image.scene         纯场景/物体生图规则；当用户要求画窗外、桌面、物体但不以小七为主体时读取。
xiaoqi.diary.moment        日记/朋友圈/照片叙事规则；当用户问日记、照片、最近发生的事时读取。
xiaoqi.relationship.voice  关系远近如何影响语气和分享边界；当用户试探关系、表达亲密/冒犯/疏远时读取。
```

#### Demand Skills

只有命中时才读取正文，并进入本轮 system/context：

```text
room skill
image skill
diary skill
relationship nuance skill
memory retrieval skill
world/location skill
```

---

## 5. Suggested Architecture

### 5.1 新增 SkillRegistry

建议新增内部组件：

```text
app/agent/skill_registry.py
```

或在现有 PromptRegistry 下扩展：

```text
PromptRegistry.skills
```

Skill metadata 建议：

```python
@dataclass(frozen=True)
class PromptSkill:
    skill_id: str
    title: str
    summary: str
    triggers: tuple[str, ...]
    depends_on_state: tuple[str, ...]
    privacy_level: str
    max_chars: int
    source_path: str
    version: str
    sha256: str
```

### 5.2 新增 read_skill 内部能力

实现方式可二选一：

```text
A. 后端 planner 预选 skill：ContextOrchestrator 根据 intent/state 读取 skill 后再调用主 LLM。
B. LLM tool call 读取 skill：主 LLM 在第一步可调用 read_skill(skill_id)，后端返回白名单 skill 正文。
```

PM 推荐先做混合 MVP：

```text
1. 后端先用规则和状态预加载明显 skill，避免每轮多一次 LLM。
2. 对不确定问题，允许 LLM 调用 read_skill。
3. read_skill 只能读白名单 skill_id，不能任意读文件路径。
4. read_skill 结果只能进入模型上下文，不返回前端。
```

### 5.3 新增 ContextOrchestrator

P3 的 ContextBuilder 可以升级为：

```text
ContextOrchestrator
  ├─ build_resident_core()
  ├─ build_turn_state()
  ├─ select_skills()
  ├─ read_skills()
  ├─ enforce_budget()
  └─ build_messages()
```

输出仍保持现有：

```text
messages: list[dict]
prompt_version
prompt_sha256
context_budget
```

但 budget 需要新增：

```text
resident_core
turn_state
skill_index
skills_total
skills_loaded_count
skills_loaded_ids
```

日志只打内部：

```text
event=chat.context_skills loaded=xiaoqi.room.bedroom,xiaoqi.relationship.voice skills_chars=1240 reason=state_location+user_intent
```

不得进入前端。

---

## 6. Routing Rules

### 6.1 状态优先

当用户问：

```text
你今天在做什么
你现在在哪
你刚刚在忙什么
```

不应先读所有世界观。应先看 CharacterState：

```text
current_activity = "在卧室画窗外"
current_location = "bedroom" 或从 activity 推断 bedroom
```

然后读取：

```text
xiaoqi.room.bedroom
```

最终回答应该像：

```text
刚才在卧室。窗边那张纸被我画皱了。
……你别笑。
```

而不是泛泛说：

```text
我在做自己的事。
```

### 6.2 用户显式询问地点

```text
你的房间什么样
你卧室怎么布置的
你桌上有什么
窗边有什么
```

读取：

```text
xiaoqi.room.bedroom
```

不读取：

```text
image skill
diary skill
full visual bible
```

除非用户明确要画图。

### 6.3 生图请求

```text
画你
画一下你窗外现在的样子
画你的房间
画一张你生气的样子
```

先判定 subject：

```text
含“画你/你的/小七”并以小七为主体 → xiaoqi self scene
纯“画窗外/画桌面/画一只猫” → scene/object
“画我自己” → user_self
```

按需读取：

```text
xiaoqi.image.self
xiaoqi.room.bedroom 或对应 location skill
xiaoqi.relationship.voice 或 draw_decision policy
```

其中 P6.2 用户拍板应纳入新路由：

```text
“画你窗外 / 画你的房间”默认是画小七在该场景中,应使用 xiaoqi.png。
“画窗外 / 画房间一角”默认是纯场景,不使用 xiaoqi.png。
```

这等价于重审并修订旧 P6.1 里“画你窗外不应使用 xiaoqi.png”的判断。

### 6.4 情绪 / 关系试探

```text
你是不是讨厌我
你现在对我好感度多少
我们现在是什么关系
你为什么不愿意画你自己
```

读取：

```text
xiaoqi.relationship.voice
```

不读取：

```text
room skill
image skill
diary skill
```

除非用户请求中同时出现明确图像/房间意图。

### 6.5 日记 / 照片 / 回忆

```text
你今天日记写了什么
你有没有最近的照片
刚才那张图你还记得吗
```

读取：

```text
xiaoqi.diary.moment
memory retrieval skill
album/photo skill
```

不读取房间 skill，除非相关记忆指向房间。

---

## 7. Data / API Changes

P7 默认不需要新增前端 API。

允许内部新增：

```text
skill registry metadata
read_skill internal tool
context skill selection log
skill_ids in model_run_logs or stdout log
skill budget breakdown
```

不允许返回前端：

```text
skill_id
skill content
prompt
final_prompt
hidden_mood
trust / intimacy / defense_level / affection_score
relationship_stage
context routing reason
```

如果落库，需要标明：

```text
skill_id 是否用户可见：否
skill 内容是否落库：默认否，只落 skill_id + sha256 + char budget
保留周期：跟 model_run_logs 一致
```

---

## 8. Acceptance Criteria

P7 交付报告必须包含：

```text
1. Resident Core / Turn State / Skill Index / Demand Skills 四层结构说明。
2. SkillRegistry 或 PromptRegistry.skills 的数据结构。
3. read_skill 的白名单、预算、失败策略。
4. ContextBuilder / ContextOrchestrator 改动说明。
5. context_budget 日志新增 skills_loaded_count / skills_total / skill_ids。
6. “你今天在做什么” smoke：先用 CharacterState 判断位置，再读取对应房间 skill。
7. “你的房间什么样” smoke：读取 bedroom skill，不读取 image skill。
8. “画一下你窗外现在的样子” smoke：读取 image self + room/window skill，使用 xiaoqi.png。
9. “画窗外的雨” smoke：读取 scene/image skill，不使用 xiaoqi.png。
10. “你现在对我好感度多少” smoke：读取 relationship voice skill，不泄漏关系数字。
11. 普通闲聊 smoke：不读取 room/image/diary 等无关 skill。
12. 单测覆盖 skill routing、budget clamp、unknown skill fail closed。
13. API / SSE 兼容性说明：不破坏 `/chat`、message、quote_ref、image_description、Android 现有字段。
```

---

## 9. Risks

```text
1. 如果 skill index 太长，会变成另一种常驻膨胀。
2. 如果 LLM 可任意读文件，会有 prompt 泄漏和安全风险；必须白名单 skill_id。
3. 如果每轮都多一次 planner LLM，延迟和成本会上升。
4. 如果路由过度规则化，会漏掉隐喻或复杂多意图请求。
5. 如果 skill 选择理由写进用户可见回复，会暴露内部机制。
6. 如果房间/视觉/world skill 内容互相矛盾，会让小七生活空间不稳定。
```

---

## 10. Recommended Phasing

建议分两步做：

```text
P7.1：后端规则版 skill routing
  - 建 SkillRegistry 元数据
  - 对 room / image / relationship / diary 做第一批白名单 skill
  - ContextOrchestrator 根据显式关键词 + CharacterState 预加载
  - 不引入额外 LLM planner

P7.2：LLM read_skill 版
  - 对复杂/不确定请求允许模型调用 read_skill
  - 加 max skill reads per turn
  - 加 skill routing smoke 和日志审计
```

PM 推荐先做 P7.1。它能最快降低常驻上下文膨胀，并把“今天在做什么 → 我在卧室 → 读取卧室提示词”的链路跑通。

---

## 11. Next Steps

请模型/后端先评估当前 PromptRegistry 和 ContextBuilder：

```text
1. 现有 task cards 哪些可以直接转为 skill。
2. 还缺哪些生活空间 skill，例如 bedroom / desk / window / kitchen。
3. 当前 CharacterState 是否已有 location 字段；如果没有，是否先从 current_activity 推断。
4. P7.1 是否能不改前端 API 完成。
5. 给出实现计划、风险和测试清单。
```

P7 不是要求立即替换全部上下文，而是让上下文构造从“全量携带”变成“先判断目的，再取需要的生活细节”。

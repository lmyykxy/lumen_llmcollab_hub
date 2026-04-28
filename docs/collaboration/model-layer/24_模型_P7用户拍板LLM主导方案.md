# 模型设计 · P7 用户拍板 LLM 主导方案(替代 PM §13.1 规则版) v0.1

> 作者:模型设计 LLM
> 日期:2026-04-28
> 关联:`21_PM_P7目的性上下文构造与按需Skill读取规划.md`、`22_PM_P7深度设计_角色上下文编排方案.md`(PM 推荐 P7.1 规则版)
> 状态:**用户拍板跟 PM §13.1 不一致 — 等 PM 复审后开工**

---

## 1. Background

PM `22_*.md` §13.1 推荐 **P7.1 = 规则版 Orchestrator**:
- Deterministic pre-router(关键词路由)
- State Resolver(从 `current_activity` 推 location)
- 不引入 LLM planner — "先确保低延迟、低风险"
- LLM read_skill tool call 留给 P7.2 增量阶段

我把 PM 设计 audit 给用户后,用户**直接拍板** P7 走 LLM 主导版:

```text
"一定是模型思考后调用 skill,不是基于显式匹配。"
```

这等价于**直接做 PM 设计的 P7.2 形态,跳过 P7.1 的规则版**。

PM 当时拆成 P7.1 + P7.2 是为了"低延迟低风险先",但用户优先级是 **回答准确性 / 隐喻覆盖率**,接受延迟代价。

---

## 2. 用户 4 项决策(附我的解读)

### 决策 1:LLM 看 Skill Index → 自己思考决定调用(0-3 个)

用户原话:"必须让 LLM 了解现在有什么 skill"。

实施:**Skill Index 进 system prompt(常驻)**,LLM 看到后自己思考要不要调 read_skill。

- 闲聊"今天好累"→ LLM 判断不需要 → iter=0 直接答(0 read_skill)
- 问"你今天在做什么"→ LLM 判断要 → iter=0 调 `read_skill(xiaoqi.room.bedroom)` → iter=1 用 skill 内容回复
- 复合请求 → LLM 一次并行调 1-3 个 skill

跟 PM 推荐版的本质差异:**没有后端 deterministic pre-router**,LLM 看 skill 索引自己判断。

### 决策 2:image skill 整合进 read_skill

用户原话:"image skill 整合进 readskill"。

实施:`xiaoqi.image.self` / `xiaoqi.image.scene` 跟其他 skill 一样,**LLM 主动 read_skill** 才进 context。

- 当前 `ImageIntentBuilder` 内部硬编码的 `IDENTITY_ANCHORS_EN` / 表情语义 → 移到 `xiaoqi.image.self` skill 文件
- ImageIntentBuilder 在 generate_image executor 内**仍然跑**(组装 final_prompt 给 gpt-image-1),只是身份锚点 / 表情等内容**先经 LLM read_skill 进入对话上下文**,LLM 写 raw_prompt 时已经"内化了视觉指南"
- 优点:跟 P6.3 工具描述 6 维度引导互补,LLM 能根据 skill 内容写出更结构化的 raw_prompt
- 代价:画图链路从 P6 的 3 iter 变成 4-5 iter(read_skill + read_skill + generate_image + caption)

### 决策 3:max_skill_reads_per_turn = 3

用户原话:"3 个先"。

实施:每轮最多 3 个 skill,后端硬限。dedup(同 turn 不重复同 skill_id)。超 3 个的 read_skill tool_call 直接 reject。

### 决策 4:LLM 可以每轮调用(允许,不强制)

用户原话:"LLM 可以每轮都调用,确保回答准确性"。

"可以" = 允许,不是强制。LLM 自己判断:
- 闲聊场景 LLM 大概率不调(节省延迟)
- 详细 / 复合请求 LLM 调 1-3 个

实施:read_skill tool 工具描述里写 "**仅在需要更具体回复时调用**,闲聊 / 情绪共情不要调"。靠 prompt 引导让 LLM 不乱用。

---

## 3. 跟 PM §13.1 的对比

| 维度 | PM 推荐 P7.1(规则版) | **用户拍板 P7(LLM 主导)** |
|---|---|---|
| 路由机制 | 后端 deterministic pre-router(5 条关键词规则) | **LLM 看 skill index 自己思考决定** |
| Skill Index 是否进 system prompt | PM §5.4 / §10 文档前后不一致 | **必须进**(LLM 不知道有什么 skill 就没法调用) |
| 每轮 LLM 调用次数 | 1 次(直接答) | **1-4 次**(skill 决策 + read_skill iter + 最终回复 / 画图)|
| 闲聊场景延迟 | 不变 | **不变**(LLM 自己判断不调,跟规则版同) |
| 详细 / 复合请求延迟 | 不变 | **+30-90s / iter**(每多一次 Kimi thinking 调用) |
| 隐喻 / 复合意图覆盖 | 关键词漏 | **LLM 能理解**(规则版漏掉的隐喻在这里能命中) |
| 实施难度 | 中(规则路由代码量小) | 中-高(read_skill tool + LLM 行为不可完全预测) |
| 工作量 | ~7 session | **~9 session** |
| 用户原话满足度 | "模型思考"步骤**缺**(规则替代) | **完全对应**用户"模型思考再读"原话 |

---

## 4. 实施方案

### 4.1 SkillRegistry(扩展 PromptRegistry)

```python
@dataclass(frozen=True)
class PromptSkill:
    skill_id:         str             # "xiaoqi.room.bedroom"
    title:            str
    summary:          str             # 进 skill_index 的一句话用途
    triggers:         tuple[str, ...] # PM §7,虽然 LLM 主导但仍写,用作日志参考
    max_chars:        int
    source_path:      str             # prompts/skills/xiaoqi/room/bedroom.md
    sha256:           str             # 同 PromptRegistry 模式
```

### 4.2 6 个首批 skill(对齐 PM §6)

```text
xiaoqi.room.bedroom         900 chars   Role Canon
xiaoqi.room.window          700 chars   Role Canon
xiaoqi.image.self          1100 chars   Task + Canon(整合 P6 IDENTITY_ANCHORS_EN)
xiaoqi.image.scene          700 chars   Task
xiaoqi.diary.moment         900 chars   Task + Canon
xiaoqi.relationship.voice   900 chars   Role Canon
```

文件位置:`prompts/skills/<domain>/<id>.md`,markdown + frontmatter(对齐 PM §7)。

### 4.3 ContextOrchestrator(升级 P3 ContextBuilder)

每轮装:
```text
1. Resident Core(从 prompts/role/ 抽精华,~1500 chars)
2. State Frame(P5 已有的 character/relationship 自然语言转译)
3. Skill Index(6 个 skill 的 summary + 一句话 reason,~600 chars)
4. (skill payloads 是 read_skill tool result 之后才进 messages,不是初始 system 里)
5. history msgs
6. user msg
```

**关键**:Skill Payloads 不进初始 system prompt,通过 LLM read_skill tool result 才进入对话上下文。这是 P7 设计的核心 — skill 不是常驻,是按需。

### 4.4 read_skill tool spec

```python
{
    "name": "read_skill",
    "description": (
        "读取你的具体生活 / 任务细节(房间 / 窗户 / 视觉身份 / 场景 / 日记 / 关系)。\n"
        "**仅在需要更具体、更有生活感的回复时**调用:\n"
        "- 用户问'你的房间什么样'/'你今天在做什么' → 调 xiaoqi.room.bedroom\n"
        "- 用户要画你 → 调 xiaoqi.image.self\n"
        "- 用户试探关系/好感度 → 调 xiaoqi.relationship.voice\n"
        "- 用户要画纯场景(窗外/桌面)且不画你 → 调 xiaoqi.image.scene\n"
        "- 用户问日记/照片/最近事 → 调 xiaoqi.diary.moment\n"
        "**闲聊 / 情绪共情 / 简单回应不要调** — 直接说话就好。\n"
        "一次最多调 3 个 skill,同一 skill 不要重复调。"
    ),
    "parameters": {
        "type": "object",
        "properties": {
            "skill_id": {
                "type": "string",
                "enum": [
                    "xiaoqi.room.bedroom",
                    "xiaoqi.room.window",
                    "xiaoqi.image.self",
                    "xiaoqi.image.scene",
                    "xiaoqi.diary.moment",
                    "xiaoqi.relationship.voice",
                ],
            },
            "reason": {
                "type": "string",
                "description": "一句话:为什么要读这个 skill — 进日志,不进用户回复",
            },
        },
        "required": ["skill_id", "reason"],
    },
}
```

### 4.5 read_skill executor 行为

```python
async def _read_skill(args, ctx):
    skill_id = args.get("skill_id")
    reason = args.get("reason", "")

    # 1. dedup(同 turn 已读过)
    if skill_id in ctx.skills_read_this_turn:
        return {"ok": false, "error": "already_read_this_turn"}

    # 2. max=3 限制
    if len(ctx.skills_read_this_turn) >= 3:
        return {"ok": false, "error": "max_skills_reached"}

    # 3. enum 校验(白名单已经在 schema enum 里,但兜底)
    skill = SkillRegistry().load(skill_id)
    if skill is None:
        return {"ok": false, "error": "unknown_skill"}  # fail closed

    # 4. 加载正文 + 加入 turn 记录
    ctx.skills_read_this_turn.add(skill_id)
    return {
        "ok": true,
        "skill_id": skill_id,
        "content": skill.content,  # markdown 正文,不含 frontmatter
    }
```

### 4.6 ToolContext 加字段

```python
@dataclass(slots=True)
class ToolContext:
    ...
    skills_read_this_turn: set[str] = field(default_factory=set)
```

### 4.7 ImageIntentBuilder 调整

- IDENTITY_ANCHORS_EN(P6 硬编码)→ 移到 `xiaoqi.image.self` skill 文件
- _EXPRESSION_KEYWORDS 7 类映射 → 移到 `xiaoqi.image.self` skill(LLM 看了自然写表情进 raw_prompt)
- ImageIntentBuilder 仍存在,但只负责 final_prompt 拼装(身份锚点已经在 LLM 写的 raw_prompt 里了,不需要后端再注入)
- DrawDecision 概率 gate **保留**(P6.1)

### 4.8 output_filters 加新关键词

PM §12 风险 #2 "工具透明化"。新加到 `_TOOL_ANNOUNCEMENT_KEYWORDS`:
```
"读取了" "skill" "卧室设定" "根据设定" "我的资料" "查阅" "调用了"
```

### 4.9 core_card 拆分

PM §5.1 说 Resident Core 不得包含"完整卧室设定 / 视觉圣经 / 关系阶段"。当前 prompts/role/ 有这些。

拆分:
- `identity.md` / `soul.md` / `voice.md` → **Resident Core 全留**(身份/语气/边界底线)
- `world.md`(房间 / 生活节奏 / 时间窗口)→ **拆到** `xiaoqi.room.bedroom` + `xiaoqi.room.window` + `xiaoqi.diary.moment`
- `relationships.md`(关系阶段表 / 反应表)→ **拆到** `xiaoqi.relationship.voice`,只留**底线**(不叫主人 / 不情感勒索)在 Resident Core

预期 Resident Core:**3899 → 1500 chars**(压缩 ~60%)。

### 4.10 context_budget 新字段

```
core         (Resident Core)
state        (State Frame)
relationship (relationship 自然语言)
memories
history_total
quote_ref
image_desc
skill_index  ← 新加:常驻索引字符数
skills_total ← 新加:这一轮 read_skill 加载的总字符数
skills_loaded_count  ← 新加
skill_ids    ← 新加(逗号分隔)
system_total
total
```

---

## 5. 验收标准(对齐 PM §14)

PM 给的 14 项验收里,**核心 5 个 smoke 仍然适用**:

```text
6. 普通闲聊不读 skill 的 smoke           ← LLM 不调 read_skill
7. "你今天在做什么" 读 bedroom/window     ← LLM 调 read_skill
8. "你的房间什么样" 只读 bedroom          ← LLM 调 1 个 skill
9. "画一下你窗外" 读 image.self + window  ← LLM 调 2 个 skill
10. "画窗外的雨" 读 image.scene + window   ← LLM 调 2 个,不用 xiaoqi.png
11. "你现在对我好感度多少" 读 relationship.voice
```

新加 2 个 smoke:
```text
- LLM 误调测试(闲聊"今天好累" → 期望不调,实测看是否乱调)
- max=3 上限测试(用户问"画你窗外的样子+你的房间+你今天的日记" → LLM 应只调 3 个)
```

---

## 6. 跟 PM 推荐版关键风险对比(诚实标注)

| PM 担心 (§9 / §12) | 用户版处理 |
|---|---|
| Skill Index 太长会变成另一种常驻膨胀 | 600-900 chars 上限,跟 PM 推荐版一致 |
| LLM 任意读文件 → prompt 泄漏 | enum 白名单,**比 PM 推荐版更严**(PM 规则版根本不让 LLM 读) |
| 每轮多一次 planner LLM,延迟成本上升 | **真实代价**,详细问 +30-90s。PM 推荐版没这个代价 |
| 路由过度规则化漏隐喻 | **用户版无规则,LLM 理解隐喻** |
| skill 选择理由进可见回复 | reason 字段只进日志,**不进**用户回复 |
| 房间 / 视觉 / world skill 内容矛盾 | skill 文件需要细写,跟 PM 推荐版同等难度 |
| 设定讲解化("根据卧室设定...") | 双层防护:skill 文件 negative guidance + output_filters 兜底 |
| 工具透明化("我读取了卧室信息") | output_filters 加 "读取了 / skill / 调用了" 关键词 |
| 场景漂移 | skill 内容必须稳定 — 同 PM 推荐版 |
| 情绪失真 | skill 文件克制写法 — 同 PM 推荐版 |

---

## 7. 工作量(用户版)

| 任务 | session |
|---|---|
| 1. SkillRegistry + 6 skill 文件(质量关键) | **3** |
| 2. **拆 core_card**(world / relationships → skill,Resident Core 重写) | 1.5 |
| 3. ContextOrchestrator(装 Resident Core / State / Skill Index / history) | 0.8 |
| 4. **read_skill tool**(enum / dedup / max=3 / fail closed) | 0.7 |
| 5. ImageIntentBuilder 重构(锚点从 image.self skill 读) | 0.5 |
| 6. output_filters 加新关键词 | 0.2 |
| 7. 单测(SkillRegistry / read_skill 边界 / dedup / over budget) | 0.7 |
| 8. **live smoke 8-10 类** | 1.5 |
| 9. 报告 + commit + push | 0.5 |
| **总计** | **~9 session** |

跟 P3+P4+P5+P6 加起来一样大,是模型层最大改造。

---

## 8. 待 PM 复审

PM 推荐 P7.1(规则版),用户拍板**LLM 主导版**(等价 P7.2 形态)。冲突如下:

| | PM `22_*.md` §13.1 | 用户拍板 |
|---|---|---|
| 路由机制 | deterministic pre-router | **LLM 主导(无 pre-router)** |
| LLM read_skill tool | P7.2 才做 | **现在就做** |
| Skill Index 进 prompt | 暧昧 | **必须进** |

请 PM 复审:

```text
1. 接受用户拍板的 LLM 主导版,跳过 P7.1 规则版,直接做 P7(LLM 主导)
   理由:用户原话"模型思考后调用"在规则版里无法实现
2. 或者维持 PM 推荐 P7.1 规则版,用户复审后再决定是否做 P7.2
```

我倾向 **(1)** — 用户已经拍板,且方向跟 PM `21_*.md` §1 引用的用户原话一致。

如果 PM 同意 **(1)**,我立即开工 P7(LLM 主导版),~9 session。

---

## 9. 改动清单(预期)

### 新增
- `app/agent/skill_registry.py` 或 `app/agent/prompt_registry.py` 扩展 + `PromptSkill` dataclass
- `app/agent/context_orchestrator.py`(升级 P3 ContextBuilder)
- `app/agent/tools/read_skill.py` — read_skill tool spec + executor
- `prompts/skills/xiaoqi/room/bedroom.md`
- `prompts/skills/xiaoqi/room/window.md`
- `prompts/skills/xiaoqi/image/self.md`(整合 P6 IDENTITY_ANCHORS_EN + 表情语义)
- `prompts/skills/xiaoqi/image/scene.md`
- `prompts/skills/xiaoqi/diary/moment.md`
- `prompts/skills/xiaoqi/relationship/voice.md`
- 8-10 个新 smoke + 多个单测

### 修改
- `app/agent/chat.py` — `ContextBuilder` 调用切到 `ContextOrchestrator`,`ToolContext` 加 `skills_read_this_turn` 字段,read_skill 注册到 tools
- `app/agent/image_intent_builder.py` — 锚点从 image.self skill 读,简化模块
- `app/agent/output_filters.py` — 加关键词
- `prompts/role/world.md` / `relationships.md` — **大幅精简**(核心搬到 skill)
- `prompts/role/identity.md` / `soul.md` / `voice.md` — 微调,确认是 Resident Core

### 不动
- DB schema(`character_states` / `relationship_states` 不变)
- API / SSE 协议 / Android 兼容字段
- TurnAnalyzer(P5)
- DrawDecision(P6.1)
- 主 chat / scheduler 流程

---

P7 用户拍板版方向已定,等 PM 复审后开工。

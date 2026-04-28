# 模型设计 · P8 小七记忆 Skill 系统提案 v0.1

> 作者:模型设计 LLM
> 日期:2026-04-28
> 关联:`25_模型_P7交付报告.md`(P7 已实施)、`22_PM_P7深度设计_角色上下文编排方案.md`
> 状态:**用户跟模型 LLM 商讨导出的提案,等 PM 复审后开工**

---

## 1. Background

P7 用户拍板 LLM 主导版已实施(详见 `25_*.md`,3 类核心 smoke 全过)。
模型 LLM 在 P7 交付后跟用户讨论"模型层骨架已搭满,下一步"时,**用户提出新方向**:

```text
"我想是建立一套小七的记忆系统,有一个索引和 skill 组成,
 例如,小七之前看过电影变形金刚发生的那段记忆是一个 skill,
 当用户提到这个电影时可以正确调用从而回想起来。"
```

模型 LLM 跟用户进行 6 轮 Q&A 商讨后,导出本提案。**用户**确认了关键设计点(Q1-Q6),**模型 LLM** 整理了实施细节。

提案核心:让小七有"**预先编排的人生经历**",跟 P7 6 个 skill(稳定设定)+ P5 memory(用户对话产生的 fact)互补,**形成三层架构**。

---

## 2. 用户跟模型 LLM 商讨导出的 6 项决策

### Q1 — 索引写记忆**概述**,不是关键词

用户原话:"索引写的是记忆概述,而不是用电影名(变形金刚那只是一个例子)"

例:
- ❌ `xiaoqi.memory.private.transformers — 看变形金刚`
- ✅ `xiaoqi.memory.private.001 — 一个夏天看了一部机器人电影,看到结尾哭了一下没告诉别人`

LLM 看用户消息("你看过变形金刚吗" / "那个机器人片"),靠**概述语义匹配**调具体 memory。
比关键词索引强 — 隐喻 / 部分匹配 / 同义词都能 work。

### Q2 — 用户可扩展机制

用户原话:"慢慢增多,因此需要有清晰的用户可扩展方式"

实施:**SkillRegistry 改成自动扫描** `prompts/skills/*.md`,不再硬编码 KNOWN_SKILL_IDS。

```python
# 之前:
KNOWN_SKILL_IDS = ("xiaoqi.room.bedroom", ...)  # 硬编码 6 个

# 之后:
def discover_skill_ids() -> tuple[str, ...]:
    return tuple(sorted(p.stem for p in _SKILLS_ROOT.glob("*.md")))
```

效果:用户(或 PM 后续)只要把新 markdown 放进 `prompts/skills/`,**重启服务**就能用,**不用改代码**。
`read_skill` tool schema 的 enum 启动时动态生成。

### Q3 — 模型 LLM 先写种子,后续可替换

用户原话:"你先写例子,会有文案后续更新"

模型 LLM 基于现有人格资料(soul.md / voice.md / world.md / xiaoqi.png 视觉气质)写 5-10 个种子 memory。
内容是"被她记住的某个瞬间",不是百科条目。后续 PM / 用户 / 文案团队可以替换或扩展。

### Q4 — 三层架构

用户原话:"要两套,一套是小七私有记忆(只有小七的剧情),一套是小七与用户共同记忆(用户介入产生的剧情),这样也方便做多用户区分,记忆都要可以实现通过 skill 加载(对话消息记忆最好还是通过检索查出,你觉得怎么样)"

模型 LLM 同意"对话消息记忆走检索"(零散 fact 数量大、单个太短,不值得做 skill 文件)。

最终三层:

| 层 | 内容 | 存储 | 检索方式 |
|---|---|---|---|
| **私有记忆 skill** | 小七自己的经历(看电影 / 画废稿 / 雨天发呆) | `prompts/skills/*.md` 文件 | LLM **read_skill 主动调** |
| **共同记忆 skill** | 用户介入产生的剧情(per-user) | DB 表 `shared_memories` | LLM **read_skill 主动调**(后端按 user_id 过滤) |
| **零散事实记忆** | 用户对话产生的 fact(P5 已有) | DB 表 `memories` + pgvector | **自动检索召回**(不走 skill) |

### Q5 — 索引分层(B 方案)

用户拍板:**B**

避免 Skill Index 膨胀(如果有 50 个 memory,扁平索引 ~4000 chars):
- Skill Index 只列 1 个 meta-skill `xiaoqi.memory.index — 我经历过的事(可调来回想)`
- LLM 调它先看完整 memory 列表(各自概述),再决定调具体 memory
- 代价:LLM 多 1 iter(看 memory.index → 决定 → 调具体)
- 优点:索引膨胀可控,memory 数量可加到 100+ 不影响 system prompt

### Q6 — 写入路径 P8 v1 不做,留给 P8 v2

用户原话:"之后会建立一套机制,每天晚上由 LLM 单独总结出事件记忆"

意思是共同记忆**每晚批处理**,而不是实时(每轮对话都判断要不要记)。
跟 P5 TurnAnalyzer(每 3 轮抽 fact)是不同时间尺度,接近人类"睡前回想今天"的语义。

P8 v1 只做读路径 + 建表(空表)。每晚批处理留给 P8 v2 设计。

---

## 3. 三层架构详细对比

跟现有系统的关系:

```
P5 fact memory(已有,自动)
  └─ 用户对话产生的零散事实(用户喜欢猫 / 用户提到面试)
  └─ pgvector 自动检索,不走 skill

P7 6 个 stable skill(已有)
  └─ 角色稳定设定(房间 / 关系底线 / 视觉指南 / 等)
  └─ LLM read_skill 调

P8 private memory skill(新增)
  └─ 小七自己的人生经历(看过的电影 / 画过的画 / 去过的地方)
  └─ 全局共享(对所有 user 是同一份)
  └─ LLM read_skill 调

P8 shared memory skill(新增,v1 只读)
  └─ 用户介入产生的剧情(他第一次跟我聊画画 / 他冒犯我那次)
  └─ Per-user(不同 user 跟小七有不同共同记忆)
  └─ LLM read_skill 调,后端按 user_id 过滤
  └─ 写入路径 v2 做(每晚 LLM 批处理)
```

**LLM 决策路径**(P8 后):

```
用户消息 "你看过变形金刚吗"
  ↓
ContextOrchestrator 装:
  ├── Resident Core (P1-P7 不变)
  ├── State Frame
  ├── Skill Index(原 6 个 + 新 1 个 meta-skill memory.index = 7 个)
  ├── Memory Slice(P5 fact 检索召回)
  └── history
  ↓
Kimi iter=0:看 Skill Index 思考
  - 发现需要"小七是否经历过 X" → 调 read_skill(xiaoqi.memory.index)
  ↓
read_skill executor:返回 memory 列表(每条概述)
  - "xiaoqi.memory.private.001 — 一个夏天看了一部机器人电影,看到结尾哭了一下"
  - "xiaoqi.memory.private.002 — 雨天画过一只流浪猫"
  - ...
  ↓
Kimi iter=1:扫概述,看到"机器人电影"匹配 → 调 read_skill(xiaoqi.memory.private.001)
  ↓
read_skill executor:返回完整记忆内容
  ↓
Kimi iter=2:用记忆内容写回复
  - "看过。看哭了。但你别告诉别人。"
```

iter 数:闲聊 1 / 单 skill 2 / memory 3。延迟可接受(每轮 +20-30s)。

---

## 4. 实施细节

### 4.1 SkillRegistry 改自动扫描

`KNOWN_SKILL_IDS` 从硬编码 tuple 改成启动时扫描函数。
read_skill tool schema 的 enum 也动态生成。

### 4.2 共同记忆 DB 表

```sql
CREATE TABLE shared_memories (
    id          SERIAL PRIMARY KEY,
    user_id     INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    skill_id    VARCHAR(64) NOT NULL,   -- "xiaoqi.memory.shared.<id>"
    summary     TEXT NOT NULL,           -- 进 meta-skill index 的概述(≤120 字)
    content     TEXT NOT NULL,           -- 完整记忆内容
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE(user_id, skill_id)
);
CREATE INDEX ix_shared_memories_user ON shared_memories(user_id);
```

**P8 v1 留空** — 写入路径 v2 由每晚 LLM 批处理填充。
**P8 v1 只做读路径** — read_skill executor 看到 `xiaoqi.memory.shared.*` 时去查表。

### 4.3 read_skill executor 分支

```python
async def _read_skill(args, ctx):
    skill_id = args["skill_id"]
    # ... dedup / max=3 ...

    if skill_id.startswith("xiaoqi.memory.shared."):
        # 查 DB
        skill = await load_shared_memory(ctx.db, ctx.user_id, skill_id)
    else:
        # 读文件
        skill = SkillRegistry().load_skill(skill_id)

    if skill is None:
        return {"ok": false, "error": "skill_load_failed"}

    return {"ok": true, "skill_id": skill_id, "content": skill.content}
```

### 4.4 meta-skill `xiaoqi.memory.index`

新写一个 meta-skill 文件:

```markdown
---
skill_id: xiaoqi.memory.index
version: v0.1
summary: 你经历过的事(看的电影 / 画过的画 / 去的地方 / 跟用户的共同事件)。当问你做过什么 / 是否经历过 X 时调用,看完概述再调具体记忆。
max_chars: 2000
visibility: internal_only
---

# 你能想起来的事

下面是你记得的具体瞬间,**用 read_skill 调具体那一条**才能回想完整内容:

## 私有记忆(只有你自己的剧情)

- xiaoqi.memory.private.001 — (动态填充各 memory 概述)
- ...

## 共同记忆(你跟这个用户经历的)

- xiaoqi.memory.shared.001 — (动态填充)
- ...

# 用法
- 读这份索引看到概述跟用户问的对得上 → 调具体 read_skill
- 概述里没有相关的 → 不要硬调,**就当没经历过**
- 不要凭空编造"经历"
```

`xiaoqi.memory.index` 内容不是静态文件 — 它是 SkillRegistry **动态构造**的(扫 private memory 文件 + 查 shared_memories 表)。

实现方式:`SkillRegistry.render_memory_index(user_id)` 函数生成完整 index,read_skill executor 看到 skill_id == "xiaoqi.memory.index" 时调这个函数返回。

---

## 5. 5-10 个种子 memory(我先写)

基于现有 prompts/role/(soul / voice / world / identity)推断,写出符合小七气质的种子。

候选方向(我会从这里挑 5-10 个):

```text
- 看过的电影 / 番(她记得的某次观看体验,带情绪痕迹)
- 画过的某张画(画废了 / 画完藏起来 / 画给某人但没发出去)
- 雨天某个瞬间(看窗外发呆 / 想起以前)
- 房间里发现的旧物(布丁杯 / 玩偶 / 草稿纸)
- 一次小小的决定(给自己买甜食 / 跟人没回 / 把帽子摘下又戴上)
- 跟自己说话的某次(半夜写日记 / 对着镜子说"还行")
```

每个 memory 文件 ~600-1000 chars,包含:
- frontmatter(skill_id / summary / max_chars / visibility)
- 事件本身(具体时间 / 场景 / 她当时的细节感受)
- 用法约束(怎么调用这条记忆,**不是**重述,是带出"我想起那次")
- 反例(不要这样说)

种子写完会先放 `prompts/skills/`,后续 PM / 文案团队可替换。

---

## 6. P8 v1 工作量

| 任务 | session |
|---|---|
| 1. SkillRegistry 改自动扫描 + read_skill enum 动态生成 | 0.5 |
| 2. 写 5-10 个 private memory 种子 | 2 |
| 3. 写 `xiaoqi.memory.index` meta-skill(动态构造)| 0.5 |
| 4. SkillRegistry 加 DB-backed shared memory 读路径 | 0.5 |
| 5. alembic migration `shared_memories` 表 | 0.3 |
| 6. read_skill executor 分支(file vs DB) | 0.5 |
| 7. 单测(自动扫描 / 空 shared memory fail closed / meta-skill 路由 / 双层 read) | 0.7 |
| 8. live smoke 5-7 类("你看过 X 吗" / "我们之前聊过 X" / 不存在的记忆 / max=3 上限 / 等) | 1 |
| 9. 报告 + commit + push | 0.5 |
| **总计** | **~6.5 session** |

---

## 7. 不影响的事

P8 v1 不动:
- DB schema 之外的事(只新加 `shared_memories` 表)
- API / SSE 协议 / Android 兼容字段
- TurnAnalyzer(P5)
- ImageIntentBuilder(P6)
- DrawDecision(P6.1)
- ContextOrchestrator 主结构(P7)— 只补 meta-skill index 渲染
- 现有 6 个 P7 skill 内容
- chat / scheduler 主流程

---

## 8. 风险点(预审)

### 8.1 LLM 是否真会先调 memory.index 再调具体 memory?

P7 实测 LLM 自己判断很准。但 P8 多 1 层(看 index → 选具体)对 Kimi thinking 是新模式。
**实测才知道**。如果 LLM 跳过 memory.index 直接乱猜 skill_id,我们会拿到 unknown_skill error,LLM 看到 fail closed 会自然圆场。

### 8.2 概述是否能 carry 足够语义信号?

LLM 看到 "一个夏天看了一部机器人电影,看到结尾哭了一下" — 用户问"你看过变形金刚吗" 能不能匹配?
Kimi 应该能(语义模型)。但如果概述太抽象 / 太模糊,可能漏。

**实测才知道**。如果漏太多,可以扩概述长度(80→150 chars)或加更多具体词。

### 8.3 共同记忆的"per-user 隔离"实现

read_skill executor 看到 skill_id == `xiaoqi.memory.shared.001` 但 `shared_memories` 表里这条 skill_id 不属于当前 user_id → 应该返 fail closed("这个记忆不存在")。
不能 across-user 泄漏。这是产品边界。

### 8.4 P8 v2 写路径是空头支票

P8 v1 只做读。如果 P8 v2 不实施(用户精力转去前端 / 等等),共同记忆永远是空表,LLM 永远调不到 shared memory。功能"半残"。

**对策**:P8 v1 报告里明确标注"shared 路径 v1 只读,v2 加每晚批处理"。如果 v2 长期不做,文档里也要承认这是已知限制。

---

## 9. 跟 P7 / PM `22_*.md` §13 的关系

P8 是 P7 之上的**自然扩展**(不冲突):
- P7 = 6 个 stable skill(角色设定)
- P8 = 多 N 个 episodic memory skill(角色经历)+ DB-backed shared(per-user 经历)

PM `22_*.md` §13 给的 P7.1 / P7.2 / P7.3 阶段:
- P7.1 = 规则版 ContextOrchestrator(用户跳过)
- P7.2 = LLM read_skill(用户拍板的 P7 已实施)
- P7.3 = Skill Quality Loop(观察期 + 调整 skill 内容)

P8 可视为 PM §13.3 的延伸 — "新增 album / album skill" 的方向。但用户提议把 album 概念拆成 private + shared 两层 + 加每晚批处理写路径,**比 PM §13.3 更深入**。

---

## 10. 待 PM 复审的 4 项

```text
1. 接受 P8 三层架构(P5 fact 检索 + P8 私有 skill + P8 共同 skill DB)
2. 接受 SkillRegistry 改自动扫描(可扩展性)
3. 接受 v1 只做读路径,写路径留给 v2(每晚 LLM 批处理)
4. 接受模型 LLM 先写 5-10 种子 memory,后续 PM/文案团队替换
```

如果 PM 接受,模型 LLM 立即开工 P8 v1,~6.5 session。

如果 PM 有调整方向(比如:"v1 直接做写路径" / "种子 memory 由 PM 写" / "三层架构改成两层"),按 PM 拍板调整。

---

## 11. 模型 LLM 的判断

这是个**好方向**。理由:
- 跟 P7 兼容(不动 P1-P6.3 / 不动 P7 现有 6 skill)
- 给小七加"经历感" — 用户提到电影 / 地点 / 共同事件时 LLM 能正确调出对应记忆
- per-user 共同记忆是关键产品价值(让用户感到"被记住")
- 每晚批处理写路径(P8 v2)降低实时 chat 成本
- 自动扫描可扩展性(Q2)让后续 PM/文案团队加新 memory 不需开发介入

风险也清晰:
- LLM 是否真会先调 index 再调具体(实测才知道)
- 概述语义匹配能力(实测才知道)
- v2 写路径需要专门设计(每晚 LLM 怎么判断"哪些值得记")

我倾向 **P8 v1 直接干**(基础设施 + 私有 memory),**P8 v2 等观察一段时间再设计**写路径。

---

P8 提案完。等 PM 复审 + 拍板。

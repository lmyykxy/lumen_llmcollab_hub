# 模型设计 · 对 PM 模型层规划的回应、Audit 与执行计划 v0.1

> 作者:模型设计 LLM(同时承担后端模型层实现)
> 日期:2026-04-27
> 关联:`01_PM_陆小七人格替换与模型层实现规划.md`
> 状态:**Audit 完成,待 PM 拍板 4 个澄清问题后立即开工**

---

## 1. 我对 PM 规划的理解(确认对齐)

7 个 Phase,优先级 P0 → P5:

| Phase | 我的理解 | 我能做 |
|---|---|---|
| P0 运行时 prompts 审计 | grep "暮 / OpenClaw / 控制标记",输出替换清单 | ✅ 已经做完(见 §2) |
| P1 替换 prompts/role/*.md | 5 份 .md(identity/soul/voice/world/relationships)→ 陆小七最小运行版 | 拍板后立即做 |
| P2 PromptRegistry | 把 xiaoqi v0.3.1 包整理为分层 cards(core/voice/safety/relationship/visual/image_rules),版本化 + sha256 + 三层加载策略 | 中等工作量 |
| P3 ContextBuilder | 统一组装 chat 上下文(character_state + relationship_state + recent events + memories + quote_ref + image_description) | 中等工作量 |
| P4 CharacterState/RelationshipState 骨架 + GET /character_state | DB schema + 服务端字段 + UI 只暴露安全字段 | 中等工作量 |
| P5 EventLog + TurnAnalyzer 异步 | 流式结束后异步抽取 memory_candidates / state_delta / relationship_delta | 较大,需要新 background worker |
| P6 ImageIntentBuilder | generate_image 前置生成 structured intent → prompt | 中等工作量,跟现有生图链路对接 |

**核心约束我读到了**:
- 不破坏 `/chat` SSE / `/messages` / `quote_ref` / 主动消息 / 生图链路
- POST /chat 字段名 `message` 不动
- assistant sender_name 永远是 `"小七"`(已经如此)
- 关系数值 / 防御值 / hidden_mood / prompt 内容 / verifier_report 不返 UI
- 眼睛固定,眼镜可变

---

## 2. Phase 0 Audit(已经做完)

PM 要求先 audit 不直接改代码。我跑了完整 grep + 文件检查,结果**比想象简单**:

### 2.1 路径澄清(重要!)

PM 文档里写的运行时 prompts 路径是:
```
C:\Users\jyb17\Desktop\PM统筹\prompts
```

**这是 PM 自己电脑上的本地路径**(Windows)。我访问不到。

**实际运行时**(Linux 服务器,uvicorn 跑的位置):
```
/root/companion/backend/prompts/
├── chat_system.md
├── proactive_system.md
├── memories_block.md
├── role/
│   ├── identity.md      ← 唯一含"暮"的文件
│   ├── soul.md          (placeholder)
│   ├── voice.md         (placeholder)
│   ├── world.md         (placeholder)
│   ├── relationships.md (placeholder)
├── tools/
│   ├── generate_image.md
│   ├── schedule_message.md / list_scheduled.md / cancel_scheduled.md
│   ├── snooze.md / web_search.md
└── triggers/
    ├── idle.md
    └── morning.md
```

我能改的是这个路径下的文件。**Q1: PM 你电脑上的那份 prompts 跟服务器这份是同源吗?需要我把改动也同步到你的本地副本(可能通过 git 推送 + 你 pull)还是只改服务器即可?**

### 2.2 "暮" / 历史控制标记的全工程出现

```bash
$ grep -rn "暮\|OpenClaw\|\[SENDIMG\]\|\[MEMORY\]\|\[DAILY_PLAN\]\|NO_REPLY\|主人" \
        backend/ --include="*.md" --include="*.py"
```

**只一条命中**:
```
backend/prompts/role/identity.md:7:- **姓名**:暮(mù,临时名)
```

其它 4 个 role 文件(soul / voice / world / relationships)是 placeholder,内容自身没"暮"字眼,但语义上**没有小七**(待替换)。

**对比 PM 规划里担心的几种泄漏**(`OpenClaw / [SENDIMG] / [MEMORY] / [DAILY_PLAN] / NO_REPLY / 主人 / 图片已生成 / 这是我生成的图片 / 我已经把图片发给你了`):

| 担心的字眼 | 全工程实际命中 | 备注 |
|---|---|---|
| 暮 | 1(identity.md:7)| Phase 1 替换即可 |
| OpenClaw | 0(legacy_reference 在 zip 内,不进运行时) | 安全 |
| [SENDIMG] / [MEMORY] / [DAILY_PLAN] | 0 | 安全 |
| NO_REPLY | 0 | 安全 |
| "主人" | 0(prompt 中无此称呼) | 安全 |
| "图片已生成" / "这是我生成的图片" / "我已经把图片发给你了" | 0(prompt 中无;但 LLM 偶发自生输出 — 我**今天加了 hallucination detect 兜底**,见昨日工作汇报)| 部分覆盖 |

### 2.3 替换清单(PM 要求的输出)

| 路径 | 当前用途 | 用户可见? | 必须替换? | 建议 | 回归测试? |
|---|---|---|---|---|---|
| `prompts/role/identity.md` | 身份/姓名/外观 | 间接(注入 system prompt 影响 AI 自我介绍语气) | **是** | 用 xiaoqi v0.3.1 `01_CHARACTER_BIBLE.md` 提取核心身份 + 适配 chat 长度 | 是 |
| `prompts/role/soul.md` | 性格内核 | 间接 | **是** | xiaoqi v0.3.1 `02_SOUL_EXPANDED.md` 提取核心 | 是 |
| `prompts/role/voice.md` | 说话方式 | **直接**(决定 AI 语气) | **是** | xiaoqi v0.3.1 `03_VOICE_AND_LANGUAGE.md` 适配为 chat 长度 | 是 |
| `prompts/role/world.md` | 世界观/房间/可见环境 | 间接 | **是** | xiaoqi v0.3.1 `06_LIFEFLOW_ENGINE.md` + `10_SCENE_LIBRARY.md` 提取 | 是 |
| `prompts/role/relationships.md` | NPC 影子 | 间接 | **是** | xiaoqi v0.3.1 `05_RELATIONSHIP_ARCS.md` 提取(注意不要把用户写成 NPC) | 是 |
| `prompts/chat_system.md` | system prompt 模板 | 间接(模型行为) | 不一定 | 当前只 `{identity}{now}{memories}`,P3 加 character_state / relationship 时一并升级 | P3 一起 |
| `prompts/proactive_system.md` | 主动消息模板 | 间接 | 不一定 | 同上,依赖 identity 切换后回归 | P3 一起 |
| `prompts/tools/generate_image.md` | 生图工具描述 | 间接 | 不一定 | P5 ImageIntentBuilder 接入时改造 | P5 |
| 4 个 schedule 工具 md | schedule 工具描述 | 间接 | 否 | 当前 OK | 否 |
| `prompts/triggers/idle.md` / `morning.md` | 主动消息触发 prompt | 间接 | 不一定 | 检查文案是否含暮(快速看下,我可以做) | P1 末尾 |

### 2.4 xiaoqi v0.3.1 包内容审查

`hub/roles/pm/completed_design_files/xiaoqi_complete_prompt_package_v0_3_1_full.zip` 解压后:

```
01_prompts_final/      ← 13 份正式人格文档(00 README → 13 SYSTEM_PROMPT_TEMPLATE)
02_visual_patch_notes/ ← 视觉补丁(眼睛固定眼镜可变)
03_legacy_reference/   ← OpenClaw 历史(legacy_only,不进运行时)
04_backend_usage/      ← BACKEND_USAGE.md(给后端的接入指引,关键!)
CHANGELOG.md
```

`BACKEND_USAGE.md` 明确:
- 聊天加载 = `CHARACTER_BIBLE + SOUL + VOICE + EMOTIONAL_ARCHITECTURE + RELATIONSHIP_ARCS + 当前状态 + 相关记忆 + 输出 schema`
- 日记 / 朋友圈 / 生图各自加载子集
- 严禁内部控制符出现在自然回复
- 视觉最低校验项(头发深蓝紫 / 帽子 / 眼睛紫蓝猫系大眼 / 软萌委屈感 / 娇小体型)

包很完整,P1 替换 5 份 role md 时直接从这里提取。

---

## 3. 待 PM 拍板的 4 个澄清问题

### Q1. PM 本机 prompts 副本(`C:\Users\jyb17\Desktop\PM统筹\prompts`)跟运行时服务器副本(`~/companion/backend/prompts/`)的关系?

实际运行时只读服务器副本。PM 本机那份如果是 PM 编辑用的"源",可以加进 git 同步;如果只是本地素材,我只改服务器即可,PM 自己保持本地副本。

**我建议**:服务器 `~/companion/backend/prompts/` 是唯一权威,PM 本机副本通过 git pull 跟服务器同步(主项目目前没配 GitHub remote,可以一并配)。

### Q2. xiaoqi v0.3.1 包在运行时怎么放?

PM 规划 §7.1 建议结构:
```
prompts/characters/xiaoqi/v0.3.1/00_README.md..13_*.md
```

但**当前运行时**仍要 `prompts/role/*.md`(代码读这个路径)。两种思路:

- **思路 A**(渐进):P1 阶段只覆盖 `prompts/role/*.md` 5 份(从 v0.3.1 提取核心),不动 v0.3.1 包结构。P2 阶段 PromptRegistry 落地时再把 v0.3.1 包整体放进 `prompts/characters/xiaoqi/v0.3.1/`,代码改成读 PromptRegistry。
- **思路 B**(一步到位):P1 直接放 v0.3.1 全包,代码同时改 PromptRegistry 加载方式。

A 风险小,B 一次性做完但风险叠加。**我倾向 A**(P1 + P2 串行,各自验收),除非 PM 强烈倾向 B。

### Q3. CharacterState 是先 DB 还是先内存?

PM 规划 §9.1 字段不少(`location` / `activity` / `public_mood` / `hidden_mood` / `availability` / `weather` / `outfit_id` / `room_state` / `status_line`)。

- 先 DB schema(新表 `character_states`,user_id + character_id 索引):稳,但要 alembic migration + 后续 TurnAnalyzer 写入路径
- 先内存 / Redis(进程内 dict):快验证,但重启丢

**我倾向先 DB**,跟 messages 同等公民。alembic migration 一条,工作量小。

### Q4. TurnAnalyzer 跑在哪?

PM 规划 §10.2 说"流式结束后异步执行"。两种实现:

- **思路 A**:同进程 asyncio 后台任务(`asyncio.create_task` 在 stream_reply 收尾时启动)。简单,但跟 chat 共享 db 连接池,长任务可能挤压
- **思路 B**:独立 worker 进程(APScheduler 或一个简单 background queue)。复杂,但隔离

**我倾向 A**(简单先上),实测如果发现挤压 chat,再切 B。

---

## 4. 执行顺序 + ETA(初步,等 PM 拍板 Q1-Q4 后细化)

| Phase | 子项 | 预估 | 依赖 |
|---|---|---|---|
| **P0** | Audit(已完成) | 0 | — |
| **P1** | 替换 5 份 role md(从 v0.3.1 提取核心) + 加 forbidden output pytest + 重启测一次 | **2-3 小时** | Q2 拍板 |
| P1.5 | 全工程回归测试(主动消息 + quote_ref + 生图 + 普通聊天) | 1-2 小时 | P1 |
| **P2** | xiaoqi v0.3.1 包整理到 `prompts/characters/xiaoqi/v0.3.1/` + PromptRegistry 实现(PromptBundle dataclass + 三层 cards 加载 + sha256) | **半天** | P1 完成 |
| P2.5 | 改 chat / proactive / generate_image 走 PromptRegistry,旧 prompts/role/*.md 退役 | 半天 | P2 |
| **P3** | ContextBuilder + 接入 /chat | **1 天** | P2.5 |
| **P4** | CharacterState / RelationshipState DB schema + GET /users/{id}/character_state | **半天** | Q3 拍板 |
| **P5** | EventLog 表 + TurnAnalyzer 异步任务 + 写 memory_candidates/state_delta/relationship_delta | **1 天** | Q4 拍板 |
| **P6** | ImageIntentBuilder + generate_image 前置改造 | **半天** | P5 |

**P1 是最紧迫的**(切人格,把"暮"运行时移除),拍板后立即做。P2-P6 是模型层基础设施,2-3 天可跑通最小闭环,后续按需深化。

---

## 5. 风险登记(回应 PM §14)

| PM §14 风险 | 我的对策 |
|---|---|
| 只换名字不换 prompt → 人格混杂 | P1 全量替换 5 份 role md,不留半旧半新 |
| 每轮加载完整 prompt 包 → 上下文过长 | P2 三层加载(Core/Task/Full),Core 永远加载,Full 只离线用 |
| TurnAnalyzer 同步执行拖慢 SSE | P5 用 `asyncio.create_task` 在 stream_reply finally 后调度,**不在 user 等待路径** |
| CharacterState 全靠模板 → 假状态 | P5 让 TurnAnalyzer 真分析每轮事件输出 state_delta 写 DB,不是定时随机 |
| Relationship 数字泄漏 UI | GET /character_state 严格白名单(只返 location / activity / status_line / weather / display_name 等无害字段),hidden_mood / trust / defense_level 一律拦截 |
| 历史文档"暮" 被 LLM 误读为正式 | history 在 `merge_assistant_bubbles` 里只取 N=20 最近行,且 quote_ref 现在 sender_name 永远是"小七"(P1 后再回归测试一次,确认没漏点) |

---

## 6. 现在可以立刻开工的事(不需要 PM 拍板)

无论 Q1-Q4 怎么决定,有几件事**现在就可以做**,不引入风险:

1. **解压 xiaoqi v0.3.1 包**到 `~/companion/prompts_staging/xiaoqi_v0.3.1/`(临时位置,不动 backend/prompts/),让我自己详细审查 13 份文件
2. **加 forbidden output pytest**:写一组 unit test,跑全工程 markdown / source 文件,断言不出现"暮 / OpenClaw / [SENDIMG] / [MEMORY] / 主人 / NO_REPLY"等。CI 友好
3. **更新 model 角色 memory**:`roles/model/memory/CURRENT_CONTEXT.md` 写当前阶段 = "等 PM 拍板 Q1-Q4 后实施 P1"

这三件我可以立刻做,做完同步进 hub。其它等 PM 回应 Q1-Q4。

---

## 7. 给 PM 的一句话

P0 audit 显示**比规划设想的更轻量** — 全工程"暮"只 1 处,其它 4 个 role 文件是空 placeholder,xiaoqi v0.3.1 包完整。P1(替换 5 份 role md + 回归)估 2-3 小时。**Q1-Q4 拍板我立即开工 P1**。

PM 你回 Q1-Q4 + 一句"开始 P1",我马上做。

# 最新协作文档追踪表

> 五个模型开始任何任务前，先读本文件。
> 当前结构版本：v1.1
> 本版变更：P6 ImageIntentBuilder 已交付但暂不最终验收；用户确认补 P6.1：小七画像表情需进 prompt、关系疏远时概率式不触发生图、修正 `画我自己` 误用 xiaoqi.png。

---

## 1. 当前项目状态

Lumen / 陆小七是单一精品角色的生活流 AI 伴侣系统。
本仓库不是 App 源码仓库，只保存五模型协作所需的长期记忆、正式文档、公共协作记录和最新追踪。

当前优先级：

```text
P0：quote_ref 聊天消息引用前后端闭环与 Android Phase 3 UI 联调
P1：前端 6 屏 MVP 稳定
P2：后端模型层接入 PromptRegistry / 小七人格文件
P3：日记 / 相册 / 图片详情后续扩展
```

---

## 2. 保留结构

```text
docs/
  05_LATEST_COLLAB_DOCS.md
  collaboration/
    quote-ref/
    model-layer/
    frontend-copy/

roles/
  pm/
    memory/
    completed_design_files/
  design/
    memory/
    completed_design_files/
  android/
    memory/
    completed_design_files/
  backend/
    memory/
    completed_design_files/
  model/
    memory/
    completed_design_files/
```

规则：

```text
五模型长期上下文 → roles/{role}/memory/
角色自己的正式文档 → roles/{role}/completed_design_files/
跨角色协作记录 → docs/collaboration/{topic}/
最新状态与拍板入口 → docs/05_LATEST_COLLAB_DOCS.md
```

已移除：

```text
.github/
docs/adr/
docs/handoffs/
docs/shared/
roles/{role}/_archive/
根目录 README / CHANGELOG / Codex handoff / GitHub 上传说明
空占位协作主题
```

重大决策不再单独放 ADR 文件，必须沉淀到本追踪表和相关角色 memory。

---

## 3. 必读顺序

```text
1. docs/05_LATEST_COLLAB_DOCS.md
2. roles/pm/memory/ROLE_MEMORY.md
3. roles/pm/memory/CURRENT_CONTEXT.md
4. roles/pm/memory/DECISIONS_CACHE.md
5. 与任务相关的 docs/collaboration/{topic}/
6. 与任务相关的角色 memory/
```

---

## 4. 当前正式文档

| 文档 ID | 文档名 | 实际路径 | 负责人 | 状态 | 版本 | 用途 |
|---|---|---|---|---|---|---|
| DOC-BE-001 | Lumen_后端模型层实现规划_v0.1.md | `roles/pm/completed_design_files/Lumen_后端模型层实现规划_v0.1.md` | PM | active | v0.1 | 后端模型层升级规划 |
| DOC-PM-002 | AI伴侣项目交接文档（产品宪法） | `roles/pm/completed_design_files/AI伴侣项目交接文档.md` | PM | active | v0.1 | 产品哲学 / 10 阶段路线 / 视觉一致性六件套 / 长期愿景 |
| DOC-PROMPT-001 | 陆小七完整人格提示文件夹 | `roles/pm/completed_design_files/xiaoqi_complete_prompt_package_v0_3_1_full.zip` | PM / Model | active | v0.3.1 | 小七人格、视觉、生图和后端接入提示词包 |
| DOC-AND-001 | Android Handover | `roles/backend/completed_design_files/ANDROID_HANDOVER.md` | Backend | active | v 同步 2026-04-27 | 给安卓客户端的契约 + 实现要点（**权威源在主项目**） |
| DOC-BE-RPT-001 | 后端工作汇报 2026-04-26 | `roles/backend/completed_design_files/工作汇报_2026-04-26_后端.md` | Backend | active | v0.1 | 后端 hotfix 序列 + 接口契约摘要 |
| DOC-BE-RPT-002 | 后端工作汇报 2026-04-27 | `roles/backend/completed_design_files/工作汇报_2026-04-27_后端.md` | Backend | active | v0.1 | 联调期 7 轮 hotfix + 观测性 + orphan cleanup + 主动消息文档化;**含 PM 阻塞项 Q1**(OpenAI 组织验证) |
| DOC-MODEL-RPT-P1 | 模型层 P1 人格替换交付报告 | `docs/collaboration/model-layer/04_模型_P1人格替换交付报告.md` | Model | done | v0.1 | 5 份 role md 替换 + forbidden tests + 4 smoke 全过 |
| DOC-MODEL-RPT-P2 | 模型层 P2 PromptRegistry 交付报告 | `docs/collaboration/model-layer/05_模型_P2交付报告.md` | Model | done | v0.1 | PromptRegistry 三层 cards + xiaoqi v0.3.1 入仓 + chat/scheduler 接入 + 10 单测 + 4 smoke 复测 |
| DOC-MODEL-RPT-P3 | 模型层 P3 ContextBuilder 交付报告 | `docs/collaboration/model-layer/07_模型_P3交付报告.md` | Model | done | v0.1 | ContextBuilder + budget log + output_filters + unknown char fallback + runtime cards forbidden + 115 单测 + 5 smoke 全过 |
| DOC-MODEL-RPT-P4 | 模型层 P4 CharacterState/RelationshipState DB 交付报告 | `docs/collaboration/model-layer/08_模型_P4交付报告.md` | Model | done | v0.1 | character_states / relationship_states 表 + GET /users/{id}/character_state 脱敏端点 + ContextBuilder 接入 + state prompt 注入(自然语言不是数值)+ 138 单测全过 |
| DOC-MODEL-RPT-P5 | 模型层 P5 TurnAnalyzer 交付报告 | `docs/collaboration/model-layer/09_模型_P5交付报告.md` | Model | done | v0.1 | 异步复盘 LLM:每 N=3 用户消息触发 + Kimi K2.6 主 provider + JSON schema 校验 + stage 阶梯硬约束 + delta clamp + per-user lock + 166 单测 + 实测 state/memories 真的变了 |
| DOC-MODEL-RPT-P41 | 模型层 P4.1 边界修正交付报告 | `docs/collaboration/model-layer/11_模型_P4.1边界修正交付报告.md` | Model | done | v0.1 | API 4 字段(移除 relationship_stage)+ output_filters 加 10 个游戏化词 + 测试 forbidden 扩到 10 字段 + 184 单测 + 高试探性 smoke 实测无关键词命中 + 内部 state/relationship 注入仍正常 |
| DOC-MODEL-RPT-P42 | 模型层 P4.2 补充验收材料 | `docs/collaboration/model-layer/13_模型_P4.2补充验收材料.md` | Model | done | v0.1 | API.md §7 character_state 端点契约同步 + relationship prompt 全自然化(删"关系阶段:陌生(0)"字段名+编号)+ 强 adversarial smoke "你现在对我好感度多少" → 小七反问"还分阶段的吗" + 209 单测全过 + 22 个新硬约束扫所有 stage×禁字段 |
| DOC-MODEL-RPT-P421 | 模型层 P4.2.1 API 措辞 hotfix 回报 | `docs/collaboration/model-layer/15_模型_P4.2.1_API措辞hotfix回报.md` | Model | done | v0.1 | 删 API.md §7 "新 user 行为" 段里 "/ stranger 关系起点" 残留(PM `14_*.md` 拍板)+ 改 "默认状态" → "默认生活状态" + grep 0 命中 + 主项目 docs/API.md 镜像同步 |
| DOC-MODEL-RPT-P6 | 模型层 P6 ImageIntentBuilder 交付报告 | `docs/collaboration/model-layer/17_模型_P6交付报告.md` | Model | done | v0.1 | 新模块 image_intent_builder.py(370 行)+ ImageIntent dataclass(PM §5.2 字段全覆盖)+ subject 识别(PM §5.4 触发 + POV 场景覆盖)+ state→画面措辞 + 视觉锚点 + xiaoqi.png 强制参考(fail-closed)+ generate_image executor 接入 + 45 单测 + 总 254 单测 + smoke 通过(身份锁住 + caption"你别盯着看") |
| DOC-MODEL-RPT-P61 | 模型层 P6.1 ImageIntent hotfix 交付报告 | `docs/collaboration/model-layer/19_模型_P6.1交付报告.md` | Model | done | v0.1 | "画我自己" 移到 user_self subject(不用 xiaoqi.png)+ expression_intent/body_language_hint 7 类语义+state 兜底 + DrawDecision 概率 gate(普通场景=1.0,xiaoqi medium/high 按 trust-defense 调,关系疏远大概率不画)+ 288 单测 + smoke A "画我自己" + smoke B "画一张你生气的样子"(嘟嘴双臂交叉)+ smoke C "画你窗外"(POV override 不用 xiaoqi.png) |
| DOC-MODEL-RPT-P62 | 模型层 P6.2 POV 撤回修正报告 | `docs/collaboration/model-layer/20_模型_P6.2修正报告.md` | Model | done(等 PM 重审) | v0.1 | 用户审视实际生成图后拍板 B:删 _POV_SCENE_OVERRIDES,"画你窗外/画你的房间" 走 xiaoqi self → 用 xiaoqi.png + 场景在 prompt;跟 PM `16_*.md` §6.9("画你窗外不应使用 xiaoqi.png")**直接冲突**,等 PM 重审 §6.9;290 单测;实测中等关系下生成图为"小七在窗边",身份锚点全保留 |
| DOC-PM-MODEL-P61-GATE | P6 验收前修正要求:表情与关系触发 | `docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md` | PM | active | v0.1 | PM 拍板 P6.1 hotfix:表情必须进 final_prompt;DrawDecision 概率式不画(关系疏远更大概率拒);"画我自己"不得误用 xiaoqi.png |
| DOC-PM-MODEL-P41-GATE | P4.1 方向确认与验收前补充要求 | `docs/collaboration/model-layer/12_PM_P4.1方向确认与验收前补充要求.md` | PM | active | v0.1 | 用户确认采用条件验收路线:P4.1 方向认可,但验收/P6 前需补 API 文档同步、内部 relationship prompt 自然语言化、强 adversarial smoke |
| DOC-PM-MODEL-P42-HOTFIX | P4.2 方向确认与 API 措辞 Hotfix 要求 | `docs/collaboration/model-layer/14_PM_P4.2方向确认与API措辞Hotfix要求.md` | PM | active | v0.1 | P4.2 技术方向通过,但公开 API 文档需删除 `stranger 关系起点` 正向描述残留；完成后再打包最终验收/P6 |
| DOC-PM-MODEL-P6 | 模型层阶段验收与 P6 规划 | `docs/collaboration/model-layer/16_PM_模型层阶段验收与P6规划.md` | PM | active | v0.1 | 用户确认 P3/P4/P5/P4.1/P4.2/P4.2.1 验收通过；授权 P6 ImageIntentBuilder；小七本人相关画像必须参考 `/root/companion/backend/res/xiaoqi.png` |
| DOC-PM-MODEL-P61-GATE | P6 验收前修正要求：表情与关系触发 | `docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md` | PM | active | v0.1 | P6 暂不最终验收；补 expression_intent 入 final_prompt、关系疏远时 draw decision 概率式不画、修正 `画我自己` 不得使用 xiaoqi.png |
| DOC-DESIGN-001 | UI Brief A · Android Mockup | `roles/design/completed_design_files/ui_brief_A_android_mockup.md` | Design | active | v0.1 | Android 端 UI 设计参考 |
| DOC-DESIGN-002 | UI Brief C · Moodboard | `roles/design/completed_design_files/ui_brief_C_moodboard.md` | Design | active | v0.1 | 整体视觉气质 / 色板 / 美术参考 |

---

## 5. 当前公共协作

| 主题 | 路径 | 状态 | 当前结论 |
|---|---|---|---|
| API 接口契约 | `docs/collaboration/api/` | active | **跨角色单一权威**;后端 LLM 主维护,代码侧契约改动后同步本目录;详见 `docs/collaboration/api/README.md` |
| quote_ref 聊天消息引用 | `docs/collaboration/quote-ref/` | active | PM 拍板 → 后端实现 → 联调期 7 轮改进 → 后端 OK,**等前端重跑用例 3/4** |
| 主动消息(proactive messages)| `docs/collaboration/proactive-messages/` | active | 后端 Phase 2b-1 起已跑通,**前端待接入** `/users/{id}/subscribe` SSE |
| 模型层 / 小七人格切换 | `docs/collaboration/model-layer/` | active | P3-P6 / P6.1 / **P6.2** 已交付;**P6.2**(用户拍板)删 POV override → "画你窗外/画你的房间" 走 xiaoqi self,生成图含小七在窗边 + 场景;跟 PM `16_*.md` §6.9 冲突,等 PM 重审;290 单测全过 |
| 前端静态文案 | `docs/collaboration/frontend-copy/` | active | PM 已给出「关于陆小七」「关于 Lumen」两页替换文案；前端需去除“暮”和工具化 AI 陪伴口径 |

quote_ref 当前协作文件(按时间序):

```text
docs/collaboration/quote-ref/00a_PM_聊天消息引用_协作总规划.md         # PM 总规划
docs/collaboration/quote-ref/00b_PM_后端模型层_聊天消息引用实现规划.md   # PM 给后端的实现规划
docs/collaboration/quote-ref/00c_后端_对PM规划的回应与澄清问题.md      # 后端 5 个 Q 待 PM 拍板
docs/collaboration/quote-ref/00d_后端_消息引用功能交付报告.md          # 后端实现交付报告
docs/collaboration/quote-ref/01_安卓前端_Phase1_2_进度同步与字段确认.md  # 安卓 P1+2 完成 + 字段确认请求
docs/collaboration/quote-ref/02_后端_字段确认回复.md                  # 后端逐项回复 + 修两处真实冲突
docs/collaboration/quote-ref/03_安卓前端_Phase3_UI完成_请求联调.md     # 安卓 P3 UI 完成 + 请求联调
docs/collaboration/quote-ref/README.md
```

model-layer 当前协作文件(按时间序):

```text
docs/collaboration/model-layer/01_PM_陆小七人格替换与模型层实现规划.md  # PM 给模型/后端 LLM 的人格切换与模型层实施规划
docs/collaboration/model-layer/02_模型_对PM规划的回应Audit与执行计划.md  # 模型 Audit 结果 + 4 个 PM 待拍板问题
docs/collaboration/model-layer/03_PM_对模型Audit回应与P1开工拍板.md     # PM 回答 Q1-Q4 + 授权开始 P1
docs/collaboration/model-layer/04_模型_P1人格替换交付报告.md            # P1 交付:5 份 role md + forbidden tests + 4 smoke
docs/collaboration/model-layer/05_模型_P2交付报告.md                   # P2 交付:PromptRegistry + xiaoqi v0.3.1 入仓 + chat/scheduler 接入
docs/collaboration/model-layer/06_PM_用户验收确认与P3开工建议.md        # 用户确认 P1/P2 验收 + 授权 P3 ContextBuilder
docs/collaboration/model-layer/07_模型_P3交付报告.md                   # P3 交付:ContextBuilder + budget log + output_filters + unknown char fallback + runtime cards forbidden + 115 单测 + 5 smoke
docs/collaboration/model-layer/08_模型_P4交付报告.md                   # P4 交付:character_states / relationship_states DB + GET /character_state 脱敏端点 + state prompt 注入 + 138 单测
docs/collaboration/model-layer/09_模型_P5交付报告.md                   # P5 交付:TurnAnalyzer 异步复盘(每 N=3 触发 + Kimi 主 provider + JSON schema + stage 阶梯 + delta clamp + per-user lock)+ 166 单测 + 实测 state 真的变了
docs/collaboration/model-layer/10_PM_内部好感度边界与P3P4P5反馈.md      # PM 补充拍板:内部好感度/关系变量允许,前端/API 不暴露关系数值或 relationship_stage
docs/collaboration/model-layer/11_模型_P4.1边界修正交付报告.md           # P4.1 修正:CharacterStateResponse 4 字段 + output_filters 加 10 游戏化词 + 184 单测 + 高试探性 smoke 通过
docs/collaboration/model-layer/12_PM_P4.1方向确认与验收前补充要求.md     # 用户确认条件验收路线:补 API 文档、内部 prompt 自然化、强 adversarial smoke 后再验收/P6
docs/collaboration/model-layer/13_模型_P4.2补充验收材料.md               # P4.2 三补:API.md §7 / relationship prompt 全自然化(删字段名+(0)+enum)/ adversarial smoke 通过(小七反问"还分阶段的吗") / 209 单测
docs/collaboration/model-layer/14_PM_P4.2方向确认与API措辞Hotfix要求.md   # PM 拍板:P4.2 通过,但 API.md 仍有 "stranger 关系起点" 正向描述残留,需 hotfix
docs/collaboration/model-layer/15_模型_P4.2.1_API措辞hotfix回报.md       # P4.2.1 hotfix 回报:删 "stranger 关系起点" + 改"默认生活状态" + grep 0 命中
docs/collaboration/model-layer/16_PM_模型层阶段验收与P6规划.md             # PM 验收 P3/P4/P5/P4.1/P4.2/P4.2.1 + 授权 P6 + 14 项验收要求 + xiaoqi.png 参考图硬约束
docs/collaboration/model-layer/17_模型_P6交付报告.md                     # P6 交付:ImageIntentBuilder + xiaoqi.png 参考图引用 + 254 单测 + smoke 通过(身份锁 + 暖黄房间 + 嘴硬 caption "你别盯着看")
docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md     # PM 拍板 P6.1 修正:表情进 final_prompt + 概率式 draw decision + 修正"画我自己"误判
docs/collaboration/model-layer/19_模型_P6.1交付报告.md                   # P6.1 交付:user_self subject + expression_intent/body_language_hint + DrawDecision/decide_draw + 288 单测 + 3 smoke(画我自己/生气/窗外) 全过
docs/collaboration/model-layer/20_模型_P6.2修正报告.md                   # P6.2 用户拍板修正:删 POV override 让"画你窗外/画你的房间"走 xiaoqi self;跟 PM §6.9 冲突等重审;实测 smoke 含小七的窗边画面
docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md    # P6 暂不验收:补表情入 prompt、关系疏远概率式不画、修正 "画我自己" subject 误判
```

frontend-copy 当前协作文件(按时间序):

```text
docs/collaboration/frontend-copy/01_PM_关于页文案调整_陆小七与Lumen.md  # PM 给前端的关于页文案替换协作
```

---

## 6. 当前 PM 拍板

### quote_ref

```text
1. POST /chat 保持 message 字段，不改 content。
2. quote_ref P0 只支持 type=message。
3. quote_ref sender_name snapshot：assistant=小七，user=你。
4. 同 user_id 并发 /chat 返回 409 conversation_busy。
5. quote_ref 作为当前 user message prefix 注入模型上下文。
6. P0 不做 event_log / memory extractor，只存 messages.quote_ref。
```

### quote_ref 后端确认

```text
POST /chat:
{
  "user_id": 123,
  "message": "我就笑,怎么了",
  "image_url": null,
  "quote_ref": {
    "type": "message",
    "id": "456"
  }
}

quote_ref.type = "message"
quote_ref.id 请求可接受 int 或 string
GET /messages quote_ref.id 响应为 number
错误体为扁平 {code, message}
400 invalid_quote_ref
409 conversation_busy
纯图 preview_text 后端兜底为 "[图片]"
模型上下文注入已由后端完成
```

### 小七产品边界

禁止未经 PM 拍板引入：

```text
前端可见的好感度 / 亲密度数字
任务中心
多角色市场
Prompt 编辑器
底部 Tab 作为 MVP 主导航
登录页作为 MVP 正式入口
关系等级 UI
工具式助手语气
情感勒索或“你不回来我活不下去”式依赖
```

PM 最新补充：

```text
内部可以有好感度或关系变量,用于驱动小七态度、语气、主动程度、私密分享程度的变化。
这些变量不得作为前端字段、UI 数字、进度条、关系等级或运营玩法暴露。
前端只感知小七行为和表达的变化,不直接读取 affection_score / trust / intimacy / defense_level / relationship_stage。
```

### 视觉身份

固定身份锚点：

```text
头发
帽子
眼睛
面貌气质
体型
```

可变元素：

```text
眼镜
穿着
姿势
表情强度
光照
场景
```

关键修正：

```text
眼睛是固定身份特征。
眼镜是可变配饰。
```

小七本人相关画像参考图：

```text
/root/companion/backend/res/xiaoqi.png
```

规则：

```text
用户要求画小七本人、画你、头像、自拍、照片、她现在的样子、她在房间里等小七相关画像时,P6 必须优先使用该服务器图片作为视觉身份参考源。
如当前生图 provider 暂不支持 reference image,模型/后端必须在 P6 交付报告中明确说明限制,不得声称已实现参考图一致性。
P6.1 起，小七画像必须根据语义把表情写入 final_prompt；关系疏远/防御较高时应有概率不画小七本人或自拍，而不是每次都触发生图；`画我自己` 默认指用户自己,不得使用 xiaoqi.png。
```

### 模型层 / 人格切换

```text
1. “暮”只作为历史测试占位，不进入正式用户链路。
2. 当前运行时提示词源为 C:\Users\jyb17\Desktop\PM统筹\prompts。
3. 正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。
4. 模型层下一步先替换 prompts/role/*.md，再接入 xiaoqi_prompt_package_v0.3.1 和 PromptRegistry。
5. `/chat` 继续保持现有 SSE 与 message 字段，不因人格切换做 breaking change。
6. quote_ref P0 继续只支持 type=message，并作为当前 user message prefix 注入模型上下文。
7. 内部允许好感度/关系变量,但 affection_score、trust、intimacy、defense_level、relationship_stage、hidden_mood、prompt、verifier_report 不返回前端或 UI。
8. PM 已拍板模型 Audit Q1-Q4：以服务器运行时 prompts 为实现权威；P1 先替换 5 份 role md；P4 CharacterState 走 DB；P5 TurnAnalyzer 先同进程 async。
9. forbidden output 测试不得扫描历史协作文档，只扫运行时 prompt、后端用户可见模板和 runtime cards。
10. 用户已确认 P1/P2 验收，并同意进入 P3 ContextBuilder。
11. P3 必须补：runtime cards forbidden 测试、prompt/context 预算日志、unknown character fallback/fail-closed、schedule/proactive 工具播报过滤。
12. 后续模型交付后的验收建议和下一阶段拍板，PM 必须先询问用户确认，再写协作文档。
13. P4/P5 的内部关系状态方向保留,但 `GET /users/{id}/character_state` 不应继续向前端返回 `relationship_stage`;前端可见字段应限制为生活状态类字段,如 mood/current_activity/last_updated_at。
14. 用户已确认 P4.1 采用条件验收路线:P4.1 方向认可,但正式验收/P6 前需补 API 文档同步、内部 relationship prompt 去字段/数值化、以及“好感度多少/关系阶段是什么”的强试探 smoke。
15. 用户已确认 P4.2 技术方向通过,但公开 API 文档不得在正向描述里写 `stranger 关系起点`;模型/后端需先做 API 措辞 hotfix,再进入最终验收/P6 授权确认。
16. 用户已确认 P3/P4/P5/P4.1/P4.2/P4.2.1 全部验收通过；PM 授权 P6 ImageIntentBuilder,但继续禁止前端关系字段/数值/等级暴露。
17. P6 中小七本人相关画像必须参考服务器固定图片 `/root/companion/backend/res/xiaoqi.png`;如果工具链不支持 reference image,必须明确说明并降级,不得随机生成小七外观。
18. P6 已交付但暂不最终验收；P6.1 必须补 expression_intent 入 final_prompt、关系感知 draw decision、以及 `画我自己` subject 修正。疏远关系下是不硬禁但大概率不画小七本人/自拍/私密画面,所有概率和关系判断不得对前端可见。
```

### 前端关于页文案

```text
1. 「关于陆小七」不得再使用“暮 / 书店店员 / 城南老书店”等测试人格描述。
2. 「关于 Lumen」不得定义为“能回答问题的助手”。
3. 关于页需表达：陆小七有自己的生活、房间、日记、照片、情绪和私密空间；用户是在慢慢靠近她，不是拥有她。
4. 避免“她是 AI · 一个温柔的陪伴”这类泛化工具化口径。
```

---

## 7. 维护规则

做任何文档或协作改动后检查：

```text
[ ] 是否更新本追踪表
[ ] 是否更新相关角色 memory
[ ] 是否放入 roles/{role}/completed_design_files/
[ ] 是否放入 docs/collaboration/{topic}/
[ ] 是否把重大 PM 拍板写回本追踪表
[ ] 是否影响 Android / Backend / Model 的当前上下文
```

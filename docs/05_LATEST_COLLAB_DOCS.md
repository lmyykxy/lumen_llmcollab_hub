# 最新协作文档追踪表

> 五个模型开始任何任务前，先读本文件。
> 当前结构版本：v0.6
> 本版变更：仓库收敛为最小协作结构，只保留角色记忆、角色正式文档、公共协作区和本追踪表。

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
| DOC-DESIGN-001 | UI Brief A · Android Mockup | `roles/design/completed_design_files/ui_brief_A_android_mockup.md` | Design | active | v0.1 | Android 端 UI 设计参考 |
| DOC-DESIGN-002 | UI Brief C · Moodboard | `roles/design/completed_design_files/ui_brief_C_moodboard.md` | Design | active | v0.1 | 整体视觉气质 / 色板 / 美术参考 |

---

## 5. 当前公共协作

| 主题 | 路径 | 状态 | 当前结论 |
|---|---|---|---|
| API 接口契约 | `docs/collaboration/api/` | active | **跨角色单一权威**;后端 LLM 主维护,代码侧契约改动后同步本目录;详见 `docs/collaboration/api/README.md` |
| quote_ref 聊天消息引用 | `docs/collaboration/quote-ref/` | active | PM 拍板 → 后端实现 → 联调期 7 轮改进 → 后端 OK,**等前端重跑用例 3/4** |
| 主动消息(proactive messages)| `docs/collaboration/proactive-messages/` | active | 后端 Phase 2b-1 起已跑通,**前端待接入** `/users/{id}/subscribe` SSE |
| 模型层 / 小七人格切换 | `docs/collaboration/model-layer/` | active | P1(5 份 role md 替换)/ P2(PromptRegistry + xiaoqi v0.3.1 + chat/scheduler 接入)/ P3(ContextBuilder + budget log + output_filters + unknown char fallback + runtime cards forbidden)已完成,5 smoke 全过、115 单测全过;等 PM 拍板 P4 CharacterState DB |
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
好感度 / 亲密度数字
任务中心
多角色市场
Prompt 编辑器
底部 Tab 作为 MVP 主导航
登录页作为 MVP 正式入口
关系等级 UI
工具式助手语气
情感勒索或“你不回来我活不下去”式依赖
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

### 模型层 / 人格切换

```text
1. “暮”只作为历史测试占位，不进入正式用户链路。
2. 当前运行时提示词源为 C:\Users\jyb17\Desktop\PM统筹\prompts。
3. 正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。
4. 模型层下一步先替换 prompts/role/*.md，再接入 xiaoqi_prompt_package_v0.3.1 和 PromptRegistry。
5. `/chat` 继续保持现有 SSE 与 message 字段，不因人格切换做 breaking change。
6. quote_ref P0 继续只支持 type=message，并作为当前 user message prefix 注入模型上下文。
7. 关系数值、防御值、hidden_mood、prompt、verifier_report 不返回 UI。
8. PM 已拍板模型 Audit Q1-Q4：以服务器运行时 prompts 为实现权威；P1 先替换 5 份 role md；P4 CharacterState 走 DB；P5 TurnAnalyzer 先同进程 async。
9. forbidden output 测试不得扫描历史协作文档，只扫运行时 prompt、后端用户可见模板和 runtime cards。
10. 用户已确认 P1/P2 验收，并同意进入 P3 ContextBuilder。
11. P3 必须补：runtime cards forbidden 测试、prompt/context 预算日志、unknown character fallback/fail-closed、schedule/proactive 工具播报过滤。
12. 后续模型交付后的验收建议和下一阶段拍板，PM 必须先询问用户确认，再写协作文档。
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

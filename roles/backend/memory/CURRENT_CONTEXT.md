# 当前上下文

> 角色：后端大模型  
> 用途：记录当前阶段该角色需要优先恢复的上下文。  
> 更新方式：每次重要任务结束后追加摘要。

---

## 当前阶段

```text
项目正在进行 Lumen / 陆小七 MVP 与模型层基础架构建设。
当前重点是 quote_ref 聊天消息引用功能 — 后端已完工部署，等前端 Phase 3 UI 联调。
```

---

## 最近任务

```text
1. 建立五模型协作仓库。
2. 去除各角色 collaboration_files，改为统一 docs/collaboration/。
3. 为每个模型新增 memory/ 文件夹。
4. quote_ref 功能 PM 拍板 5 个 Q（Q1-Q5）。
5. 后端实现 quote_ref 完整链路并部署（2026-04-27）：
   - DB migration messages.quote_ref JSONB（rev a8c4f0d2e7b1）
   - POST /chat 接受 quote_ref + 校验 + snapshot 生成
   - GET /messages 回显 quote_ref
   - merge_assistant_bubbles 注入 quoted_context 到 user message prefix
   - 并发保护 409 conversation_busy（同步 set check-then-mark）
   - id 接受 int|str（适配前端字符串发送，2026-04-27 安卓字段确认）
   - 错误响应体扁平 {code, message}（适配前端 parser，2026-04-27 同上）
6. 写《02_后端_字段确认回复.md》回应安卓 §1 + 同步 ~/companion/docs/API.md。
```

---

## 当前后端关键文件指针（quote_ref 相关）

```text
~/companion/backend/app/agent/quoting.py
  - QuoteRefIn / QuoteRefSnapshot
  - build_quote_snapshot
  - format_quoted_context_prefix
~/companion/backend/app/web/main.py::chat()
  - 并发保护 _BUSY_USERS
  - 扁平 JSONResponse 错误（{code, message}）
~/companion/backend/app/agent/memory/ops.py::merge_assistant_bubbles
  - 注入 quote_ref 到 user message prefix
~/companion/backend/alembic/versions/a8c4f0d2e7b1_messages_quote_ref_for_message_quoting.py
```

---

## 等待

```text
- 安卓前端 Phase 3 UI 完成后联调。
- 没有后端阻塞项。
```

---

## 2026-04-27 结构收敛

```text
仓库已收敛为最小结构：五角色 memory、五角色 completed_design_files、docs/collaboration 公共协作区、docs/05_LATEST_COLLAB_DOCS.md。
后端侧后续只维护 roles/backend/memory/ 与 roles/backend/completed_design_files/。
docs/adr、docs/handoffs、docs/shared 已移除；重大决策沉淀到最新追踪表与相关角色 memory。
```

## 2026-04-27 模型层人格切换规划

```text
新增协作文档：docs/collaboration/model-layer/01_PM_陆小七人格替换与模型层实现规划.md。

当前运行时提示词源：C:\Users\jyb17\Desktop\PM统筹\prompts。
已确认 role/identity.md 仍写“暮(mù,临时名)”，role/*.md 整体仍是临时占位。

后端下一步不应继续把“暮”作为正式 assistant 身份。正式链路统一：
- character_id=xiaoqi
- display_name=陆小七
- assistant sender_name=小七
- prompt_version=xiaoqi_prompt_package_v0.3.1

实施顺序：先替换运行时 prompts/role/*.md 并清理“暮 / OpenClaw / 控制标记”在用户可见链路的残留，再接 PromptRegistry、ContextBuilder、model_run_logs / event_log 最小骨架、TurnAnalyzer 和 ImageIntentBuilder。不得破坏现有 /chat SSE、message 字段、quote_ref P0、主动消息和 Android 兼容字段。
```

## 2026-04-27 PM 回复模型 Audit

```text
PM 已回复 docs/collaboration/model-layer/03_PM_对模型Audit回应与P1开工拍板.md。

后端/模型可以立即开始 P1：
1. 以 ~/companion/backend/prompts/ 为运行时权威。
2. 先替换 prompts/role/*.md 五份文件。
3. 暂不上 PromptRegistry，P2 再做。
4. CharacterState 后续走 DB。
5. TurnAnalyzer 后续先用同进程 async。
6. forbidden output 测试不要扫历史协作文档，只扫运行时 prompt、后端用户可见模板和 runtime cards。
```

## 2026-04-28 用户验收 P1/P2 与 P3

```text
模型/后端已交付 P1 人格替换和 P2 PromptRegistry。
用户已确认 P1/P2 验收，并同意进入 P3。
PM 已新增 docs/collaboration/model-layer/06_PM_用户验收确认与P3开工建议.md。

后端 P3 注意：
1. 不破坏 /chat SSE、message 字段、quote_ref、image_description、Android 兼容字段。
2. schedule/proactive 系统播报式话术加入硬过滤或重写。
3. prompt/version/sha256 继续记录，P3 增加 model_run_logs 或等价最小日志表。
4. unknown character fallback xiaoqi 或 fail closed，不能生成无身份助手。
```

## 2026-04-28 内部好感度边界

```text
用户补充拍板：小七需要内部好感度/关系变量,但不能给前端。

后端/API 执行边界：
1. DB 内部可以保存 affection_score 或 trust / intimacy / defense_level / stage 等关系变量。
2. 这些变量只给模型层、TurnAnalyzer、ContextBuilder、调试/后台审计使用。
3. 面向前端的接口不得返回 affection_score、trust、intimacy、defense_level、relationship_stage、prompt_summary、verifier_report。
4. P4 已交付的 `GET /users/{id}/character_state` 当前返回 `relationship_stage`,需要调整响应 schema 和测试。
5. 前端可见 character_state 仅保留生活状态类字段,例如 user_id、mood、current_activity、last_updated_at。
```

## 2026-04-28 P4.1 验收前补充要求

```text
P4.1 方向已被 PM/用户认可,但不是最终验收。后端侧需先补：
1. 同步公开 API 契约文档,确认 `GET /users/{id}/character_state` 只返回 user_id/mood/current_activity/last_updated_at。
2. 如 Android Handover 或 docs/collaboration/api 仍有 relationship_stage,必须删除。
3. 配合模型侧把内部 relationship prompt 改成自然语言,避免“关系阶段:陌生(0)”和内部数值。
4. 补强试探 smoke 与过滤检测结果。

新增 PM 文档：docs/collaboration/model-layer/12_PM_P4.1方向确认与验收前补充要求.md。
补充完成前不得启动 P6。
```

## 2026-04-28 P4.2 API 措辞 Hotfix

```text
P4.2 技术方向已被 PM 认可,但 docs/collaboration/api/API.md 仍有 `stranger 关系起点` 正向描述残留。
后端侧需做小 hotfix：
1. API.md character_state 新 user 行为只描述默认生活状态。
2. 主项目镜像 docs/API.md 同步同样改动。
3. grep 确认 `stranger 关系起点` 不再出现。
4. relationship_stage / trust / intimacy / defense_level / affection_score 不得作为返回字段或正向状态描述。

新增 PM 文档：docs/collaboration/model-layer/14_PM_P4.2方向确认与API措辞Hotfix要求.md。
hotfix 完成前不得开 P6。
```

## 2026-04-28 P6 ImageIntentBuilder

```text
用户已确认 P3/P4/P5/P4.1/P4.2/P4.2.1 验收通过,PM 已授权进入 P6 ImageIntentBuilder。

后端侧必须注意：
1. 小七本人相关画像必须参考服务器固定图片 `/root/companion/backend/res/xiaoqi.png`。
2. 该路径是服务端内部视觉身份参考源,不得作为前端字段返回。
3. P6 需要先检查该文件是否存在、可读。
4. 需要审计当前 generate_image provider 是否支持 reference image / image-to-image。
5. 如果不支持,交付报告必须明确说明限制并降级,不得随机生成一个新小七外观。
6. 不得为 P6 新增前端关系字段或暴露内部状态数值。
```

## 2026-04-28 P6.1 表情与关系触发修正

```text
P6 已交付,但 PM/用户要求先做 P6.1 后再验收。
新增 PM 文档：docs/collaboration/model-layer/18_PM_P6验收前修正要求_表情与关系触发.md。

后端侧注意：
1. generate_image 执行前或工具调用前需要承接 draw_decision / image_consent_gate。
2. 低亲密/高防御时,小七本人、自拍、正脸、私密房间请求应有概率不调用生图 provider；不是硬禁。
3. 不画时应返回自然小七口吻,不暴露概率、关系状态或内部数值。
4. 小七本人画像 final_prompt 必须带语义表情描述。
5. `画我自己` 不得使用 `/root/companion/backend/res/xiaoqi.png`。
6. 不改前端 API,不新增 relationship / probability 字段。
```

## 2026-04-28 P7 目的性上下文构造

```text
PM 新增规划：
docs/collaboration/model-layer/21_PM_P7目的性上下文构造与按需Skill读取规划.md。

后端侧注意：
1. 需要评估 PromptRegistry 是否扩展为 SkillRegistry,或新增 app/agent/skill_registry.py。
2. read_skill 必须是内部白名单能力,只能通过 skill_id 读取,不能暴露任意文件读取。
3. ContextBuilder 可升级为 ContextOrchestrator,记录 skills_loaded_count / skill_ids / skills_total budget。
4. CharacterState 需要能提供 current_location,或至少从 current_activity 推断 location。
5. P7 不改前端 API / SSE / message 字段,skill_id、prompt、routing reason 不得返回前端。
```

## 2026-04-28 P7 深度设计

```text
新增 PM 深度设计文档：
docs/collaboration/model-layer/22_PM_P7深度设计_角色上下文编排方案.md。

后端侧执行重点：
1. P7.1 先实现规则版 SkillRegistry / ContextOrchestrator,不要求每轮多一次 LLM planner。
2. read_skill 如后续实现,必须只接受 enum skill_id,不能接受 path。
3. 首批 skill 只做 bedroom/window/image_self/image_scene/diary_moment/relationship_voice。
4. context_budget 增加 skill_ids / skills_total / skills_loaded_count。
5. skill 内容、routing reason、prompt 不得返回前端或 SSE。
6. unknown skill / over budget 必须 fail closed,不能让模型编造房间设定。
```

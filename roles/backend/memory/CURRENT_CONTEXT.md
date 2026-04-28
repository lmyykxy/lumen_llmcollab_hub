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

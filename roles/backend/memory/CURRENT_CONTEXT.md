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

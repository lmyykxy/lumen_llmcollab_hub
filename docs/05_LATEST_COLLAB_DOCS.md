# 最新协作文档追踪表

> 五个大模型开始任何任务前，先读本文件。  
> 当前结构版本：v0.5  
> 本版新增：每个角色都有自己的 `memory/` 目录。

---

## 1. 当前结构规则

### 角色正式成果与记忆

| 角色 | 已完成设计文件 | 角色记忆 | 历史归档 |
|---|---|---|---|
| PM | `roles/pm/completed_design_files/` | `roles/pm/memory/` | `roles/pm/_archive/` |
| 设计 | `roles/design/completed_design_files/` | `roles/design/memory/` | `roles/design/_archive/` |
| 安卓 | `roles/android/completed_design_files/` | `roles/android/memory/` | `roles/android/_archive/` |
| 后端 | `roles/backend/completed_design_files/` | `roles/backend/memory/` | `roles/backend/_archive/` |
| 模型设计 | `roles/model/completed_design_files/` | `roles/model/memory/` | `roles/model/_archive/` |

### 统一协作区

所有协作中内容统一放：

```text
docs/collaboration/{topic}/
```

### 正式交接区

跨角色正式交接放：

```text
docs/handoffs/{from-to-or-topic}/
```

### 重大决策

放：

```text
docs/adr/
```

---

## 2. memory/ 的作用

`memory/` 不是交付文档，而是每个模型自己的长期上下文缓存。

每个角色建议包含：

```text
ROLE_MEMORY.md
CURRENT_CONTEXT.md
DECISIONS_CACHE.md
OPEN_QUESTIONS.md
WATCHLIST.md
```

开始任务前读取顺序：

```text
1. docs/05_LATEST_COLLAB_DOCS.md
2. roles/{role}/memory/ROLE_MEMORY.md
3. roles/{role}/memory/CURRENT_CONTEXT.md
4. docs/collaboration/{topic}/
```

---

## 3. 当前最新文档总表

| 文档 ID | 文档名 | 建议路径 | 负责人 | 状态 | 版本 | 用途 |
|---|---|---|---|---|---|---|
| DOC-PM-001 | Lumen_设计与安卓前端交接文档_PM.md | `roles/pm/completed_design_files/` | PM | active | v0.1 | 给设计与安卓的总体交接 |
| DOC-FE-001 | 前端安卓交付备注_基于UI设计报告.md | `roles/android/completed_design_files/` | PM / Android | active | v0.1 | 前端实现备注与禁区 |
| DOC-BE-001 | Lumen_后端模型层实现规划_v0.1.md | `roles/backend/completed_design_files/` | PM / Backend / Model | active | v0.1 | 后端模型层升级规划 |
| DOC-QUOTE-001 | 聊天消息引用前后端规划 | `docs/collaboration/quote-ref/` | PM / Android / Backend | active | v0.1 | quote_ref 前后端协作规划 |
| DOC-PROMPT-001 | 陆小七完整人格提示文件夹_v0.3.1 | `roles/model/completed_design_files/xiaoqi_prompt_package_v0.3.1/` | Model | active | v0.3.1 | 小七人格与提示文件全集 |
| DOC-VISUAL-001 | 视觉补丁：眼睛固定眼镜可变 | `roles/model/completed_design_files/visual_patch_v0.3.1/` | Model / Design | active | v0.3.1 | 小七视觉锚点修正 |
| DOC-CRITIC-001 | Consistency Critic 设计说明 | `docs/collaboration/consistency-critic/` | Model | draft | v0.1 | 一致性批判模型长期设计 |
| ADR-0001 | POST /chat 保持 message 字段 | `docs/adr/ADR-0001-use-message-field-for-chat-api.md` | PM / Backend / Android | active | v1 | API 字段拍板 |
| ADR-0002 | quote_ref P0 只支持聊天消息引用 | `docs/adr/ADR-0002-quote-ref-p0-scope.md` | PM / Backend / Android | active | v1 | 引用功能范围拍板 |

---

## 4. 最近 PM 拍板摘要

### quote_ref 功能

```text
Q1：POST /chat 保持 message 字段，不改 content。
Q2：sender_name 选 A，后端 snapshot 写 assistant=小七，user=你。
Q3：并发保护 409 conversation_busy 一并做。
Q4：quote_ref 作为当前 user message prefix 注入模型上下文。
Q5：本期不做 event_log / memory extractor，只存 messages.quote_ref。
```

---

## 5. 维护 checklist

```text
[ ] 是否更新本追踪表
[ ] 是否更新相关角色 memory
[ ] 是否放入正确目录
[ ] 是否需要从 docs/collaboration/ 转入 completed_design_files/
[ ] 是否需要复制到 docs/handoffs/
[ ] 是否需要新增 ADR
[ ] 是否影响 API.md
[ ] 是否影响 Android handover
[ ] 是否影响 Prompt 版本
[ ] 是否需要 PM 拍板
```


---

## Codex 接手文件

| 文件 | 路径 | 状态 | 用途 |
|---|---|---|---|
| AGENTS.md | `AGENTS.md` | active | Codex / Agent 仓库级行为规则 |
| CODEX_HANDOFF_PROMPT.md | `CODEX_HANDOFF_PROMPT.md` | active | Codex 新对话接手提示词 |
| GITHUB_WEB_UPLOAD_GUIDE.md | `GITHUB_WEB_UPLOAD_GUIDE.md` | active | GitHub 网页上传说明 |

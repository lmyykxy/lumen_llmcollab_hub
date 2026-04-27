# Lumen LLM Collaboration Hub

> 用途：管理 Lumen / 陆小七项目中五个大模型角色的协作、交接、拍板、成果沉淀与角色记忆。  
> 五个角色：PM / 设计 / 安卓前端 / 后端 / 模型设计。  
> 当前结构版本：v0.5  
> 本版新增：每个大模型拥有自己的 `memory/` 文件夹。

---

## 1. 当前目录结构

```text
roles/
  pm/
    completed_design_files/
    memory/
    _archive/

  design/
    completed_design_files/
    memory/
    _archive/

  android/
    completed_design_files/
    memory/
    _archive/

  backend/
    completed_design_files/
    memory/
    _archive/

  model/
    completed_design_files/
    memory/
    _archive/

docs/
  05_LATEST_COLLAB_DOCS.md
  collaboration/
  handoffs/
  adr/
  shared/
```

---

## 2. 三类角色文件夹

### completed_design_files/

放本角色已经确认、可执行、可作为交付依据的成果文件。

### memory/

放本角色的长期记忆文件。

这些文件不是正式交付文档，而是帮助该模型在后续会话中快速恢复上下文。

### _archive/

放旧版、废弃、被替代但仍需要追溯的文件。

---

## 3. 统一协作区

所有协作中内容统一放：

```text
docs/collaboration/{topic}/
```

不再使用各角色自己的 `collaboration_files/`。

---

## 4. 记忆文件使用规则

每个模型的 `memory/` 目录建议包含：

```text
ROLE_MEMORY.md          该角色长期记忆总表
CURRENT_CONTEXT.md      当前阶段上下文
DECISIONS_CACHE.md      与该角色相关的 PM 拍板摘要
OPEN_QUESTIONS.md       待确认问题
WATCHLIST.md            该角色需要持续关注的风险
```

开始新任务前，模型读取顺序：

```text
1. docs/05_LATEST_COLLAB_DOCS.md
2. roles/{role}/memory/ROLE_MEMORY.md
3. roles/{role}/memory/CURRENT_CONTEXT.md
4. 当前 topic 的 docs/collaboration/{topic}/
```
---

## Codex / GitHub 接手入口

本仓库已经包含：

```text
AGENTS.md
CODEX_HANDOFF_PROMPT.md
GITHUB_WEB_UPLOAD_GUIDE.md
```

使用方式：

```text
1. 上传本仓库到 GitHub。
2. Codex 新对话先读取 AGENTS.md。
3. 将 CODEX_HANDOFF_PROMPT.md 内容复制给 Codex。
4. 后续任务按 docs/05_LATEST_COLLAB_DOCS.md 与角色 memory 继续推进。
```

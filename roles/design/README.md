# 设计大模型工作区

## 职责

UI/UX、视觉稿、交互状态、设计 token、前端交接说明。

---

## completed_design_files/

放已经确认、可执行、可作为交付依据的文件。

---

## memory/

放该模型的长期记忆文件，用于跨会话恢复上下文。

建议每次任务开始前先读：

```text
roles/design/memory/ROLE_MEMORY.md
roles/design/memory/CURRENT_CONTEXT.md
```

---

## _archive/

放旧版、被替代、废弃但仍需追溯的文件。

---

## 协作文件放哪里？

本工程不使用各角色自己的协作文件夹。  
所有协作中内容统一放：

```text
docs/collaboration/{topic}/
```

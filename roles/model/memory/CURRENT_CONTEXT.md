# 当前上下文

> 角色：模型设计大模型  
> 用途：记录当前阶段该角色需要优先恢复的上下文。  
> 更新方式：每次重要任务结束后追加摘要。

---

## 当前阶段

```text
项目正在进行 Lumen / 陆小七 MVP 与模型层基础架构建设。
当前重点是 quote_ref 聊天消息引用功能，以及五模型协作仓库规范化。
```

---

## 最近任务

```text
1. 建立五模型协作仓库。
2. 去除各角色 collaboration_files，改为统一 docs/collaboration/。
3. 为每个模型新增 memory/ 文件夹。
4. quote_ref 功能已完成 PM 拍板，等待前后端执行。
```

---

## 2026-04-27 结构收敛

```text
仓库已收敛为最小结构：五角色 memory、五角色 completed_design_files、docs/collaboration 公共协作区、docs/05_LATEST_COLLAB_DOCS.md。
模型侧后续只维护 roles/model/memory/ 与 roles/model/completed_design_files/。

当前提示词包来源仍是 roles/pm/completed_design_files/xiaoqi_complete_prompt_package_v0_3_1_full.zip。
模型规则继续以 v0.3.1 为准：小七不是工具，眼睛固定，眼镜可变，legacy_reference 不直接加载。
```

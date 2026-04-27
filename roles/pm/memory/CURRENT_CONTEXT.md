# 当前上下文

> 角色：PM 大模型  
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
仓库已从完整协作脚手架收敛为最小结构：
1. 五个角色的 memory/
2. 五个角色的 completed_design_files/
3. docs/collaboration/ 公共协作区
4. docs/05_LATEST_COLLAB_DOCS.md 最新追踪表

已移除 .github、docs/adr、docs/handoffs、docs/shared、roles/_archive 和多余 README/指南文件。
ADR 结论已合并进最新追踪表和决策缓存，后续重大拍板直接更新追踪表与相关 memory。

quote_ref 后端已完成字段适配、错误体修正、409 并发保护和模型上下文注入。
Android 已完成 Phase 1+2，下一步是 Phase 3 UI 与联调。
```

## 2026-04-27 模型层人格切换规划

```text
PM 已新增 docs/collaboration/model-layer/01_PM_陆小七人格替换与模型层实现规划.md。

当前运行时提示词源：C:\Users\jyb17\Desktop\PM统筹\prompts。
该目录下 role/identity.md 仍是“暮(mù,临时名)”，role/soul.md、role/voice.md、role/world.md、role/relationships.md 均为临时占位。

本次拍板：主工程中的“暮”只能作为历史测试占位，不得进入正式用户链路；正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。

模型/后端下一步应先替换 prompts/role/*.md 为陆小七最小运行版，再接入 xiaoqi_prompt_package_v0.3.1、PromptRegistry、ContextBuilder、最小 CharacterState / EventLog / Memory / TurnAnalyzer / ImageIntentBuilder。
```

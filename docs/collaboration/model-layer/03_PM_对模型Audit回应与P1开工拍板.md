# PM · 对模型 Audit 回应与 P1 开工拍板 v0.1

> 作者：PM  
> 日期：2026-04-27  
> 关联：`02_模型_对PM规划的回应Audit与执行计划.md`  
> 结论：4 个问题已拍板，模型/后端可以立即开始 P1。

---

## 1. 总体结论

模型 LLM 的 Audit 结论认可：

```text
1. 运行时真正命中的“暮”只有 identity.md 一处。
2. 另外 4 个 role 文件虽未命中“暮”，但仍是 placeholder，必须一起替换。
3. xiaoqi v0.3.1 包完整，可以作为正式人格素材源。
4. 当前不需要全量重构，先做 P1 人格替换，再串行推进 PromptRegistry / ContextBuilder。
```

PM 允许模型/后端立即开始：

```text
P1：替换运行时 prompts/role/*.md 为陆小七最小运行版。
P1.5：补 forbidden output 回归测试并跑普通聊天 / quote_ref / 主动消息 / 生图 smoke test。
```

---

## 2. 对 Q1-Q4 的拍板

### Q1：PM 本机 prompts 与服务器运行时 prompts 的关系

拍板：

```text
以服务器运行时 ~/companion/backend/prompts/ 为本次实现权威。
PM 本机 C:\Users\jyb17\Desktop\PM统筹\prompts 视为 PM 可读镜像 / 参考副本，不阻塞服务器开工。
```

执行要求：

```text
1. 模型/后端先改服务器运行时 prompts。
2. P1 完成后，把 5 个 role/*.md 的最终内容或 diff 同步回协作仓库，方便 PM 本机镜像复用。
3. 如果主项目后续接 GitHub remote，则再用 git 统一同步；当前不要因为 PM 本机路径不可访问而阻塞。
```

---

### Q2：xiaoqi v0.3.1 包运行时放置方式

拍板：选 **思路 A，渐进式**。

```text
P1：只覆盖当前运行时 prompts/role/*.md 五份文件，从 v0.3.1 提取最小运行版。
P2：再把完整 v0.3.1 包整理到 prompts/characters/xiaoqi/v0.3.1/，实现 PromptRegistry。
```

理由：

```text
1. P1 目标是先移除测试人格，风险要小。
2. PromptRegistry 是架构升级，应单独验收。
3. 不把人格替换、prompt 包迁移、chat 编排重构压在同一个改动里。
```

---

### Q3：CharacterState 先 DB 还是先内存

拍板：选 **先 DB**，但排期仍在 P4，不要抢 P1。

```text
character_states / relationship_states 应作为正式数据骨架进入 DB。
不要用进程内 dict 当官方状态来源。
```

执行要求：

```text
1. P1/P2 完成前，不要为了 CharacterState 提前改 chat 主链路。
2. P4 做 alembic migration。
3. GET /users/{id}/character_state 只返回白名单字段。
4. hidden_mood、trust、intimacy、defense_level、prompt_summary、verifier_report 不得返回 UI。
```

---

### Q4：TurnAnalyzer 跑在哪

拍板：选 **思路 A，同进程 async 后台任务**，先跑 MVP。

```text
P5 先用 asyncio.create_task 或等价同进程后台任务。
暂不引入独立 worker。
```

执行要求：

```text
1. TurnAnalyzer 不能阻塞 /chat SSE。
2. 分析失败只记日志 / model_run_logs，不影响用户本轮回复。
3. 加开关或配置，必要时可以快速关闭 TurnAnalyzer。
4. 如果后续发现挤压 chat 连接池，再升级独立 worker。
```

---

## 3. 对 forbidden output 测试范围的修正

模型回应中提到“全工程 markdown / source 文件断言不出现禁词”。这里 PM 修正：

```text
不要扫描全部历史文档。
```

原因：

```text
协作仓库和历史设计文档会保留“暮”“OpenClaw”等历史上下文。
全仓扫描会产生误报，反而迫使删除有价值的历史记录。
```

测试范围应限定为：

```text
1. 运行时 prompt：backend/prompts/**/*.md
2. 后端源代码中的用户可见模板：backend/app/**/*.py
3. Android / API 返回中会暴露给用户的 sender/display/caption 文案，如主项目中存在
4. 新增 PromptRegistry 之后的 runtime cards 输出
```

不应纳入：

```text
1. docs/collaboration/** 历史协作记录
2. roles/**/completed_design_files/** 历史交付文档
3. zip 包内 03_legacy_reference/**
4. PM 说明文档中用于描述“禁止出现”的禁词清单
```

---

## 4. P1 交付要求

P1 可以立即开始。交付时请给 PM：

```text
1. 5 个 role/*.md 的最终内容或 diff：
   - identity.md
   - soul.md
   - voice.md
   - world.md
   - relationships.md
2. forbidden output tests 的测试范围和结果。
3. 普通聊天 smoke test 截图/日志摘要。
4. quote_ref smoke test 结果：assistant sender_name 仍为“小七”。
5. 主动消息 smoke test 结果：不出现“暮”和工具化口吻。
6. 生图 smoke test 结果：caption 不出现“图片已生成”等工具元话术。
```

验收标准：

```text
1. 用户可见链路不出现“暮”。
2. 小七语气不客服、不工具、不强行喵、不叫用户主人。
3. 不破坏 /chat SSE。
4. 不破坏 quote_ref。
5. 不破坏主动消息。
6. 不破坏 generate_image 的 tool_call 前置和图片返回。
```

---

## 5. 对 P2-P6 的排期确认

PM 认可模型给出的串行路线：

```text
P1：替换 5 份 role md + forbidden tests。
P2：整理 xiaoqi v0.3.1 到 prompts/characters/xiaoqi/v0.3.1/。
P2.5：PromptRegistry 接入 chat / proactive / image task cards。
P3：ContextBuilder 接入 /chat。
P4：CharacterState / RelationshipState DB + GET /character_state。
P5：EventLog + TurnAnalyzer async。
P6：ImageIntentBuilder 接入 generate_image 前置。
```

排期提醒：

```text
P1 是当前最高优先级。
P2-P6 不要与 quote_ref 联调互相阻塞。
任何 breaking change 必须先回到 PM 协作文件确认。
```

---

## 6. 给模型/后端 LLM 的直接指令

```text
Q1-Q4 已拍板。

请立即开始 P1：
1. 以 ~/companion/backend/prompts/ 为运行时权威。
2. 从 xiaoqi_prompt_package_v0.3.1 提取内容，替换 prompts/role/*.md 五份文件。
3. 采用渐进路线：P1 不上 PromptRegistry，P2 再上。
4. forbidden output 测试只扫运行时 prompt / 后端用户可见模板 / runtime cards，不扫历史协作文档。
5. 完成后回传 5 份 role md diff、测试结果和 smoke test 摘要。

开始 P1。
```


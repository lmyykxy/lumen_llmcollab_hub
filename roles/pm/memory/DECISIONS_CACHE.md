# 决策缓存

本文件记录和本角色相关的 PM 拍板摘要。  
正式依据以 `docs/05_LATEST_COLLAB_DOCS.md` 为准。

---

## quote_ref 功能

```text
Q1：POST /chat 保持 message 字段，不改 content。
Q2：sender_name 选 A，后端 snapshot 写 assistant=小七，user=你。
Q3：并发保护 409 conversation_busy 一并做。
Q4：quote_ref 作为当前 user message prefix 注入模型上下文。
Q5：本期不做 event_log / memory extractor，只存 messages.quote_ref。
```

## 2026-04-27 结构拍板

```text
仓库只保留：
1. 五角色 memory/
2. 五角色 completed_design_files/
3. docs/collaboration/ 公共协作区
4. docs/05_LATEST_COLLAB_DOCS.md 最新追踪表

不再保留 docs/adr、docs/handoffs、docs/shared、roles/_archive 和根目录交接脚手架。
重大拍板写入最新追踪表和相关角色 memory。
```

## 2026-04-27 模型层人格切换

```text
Q1：“暮”只作为历史测试占位，不进入正式用户链路。
Q2：当前运行时提示词源为 C:\Users\jyb17\Desktop\PM统筹\prompts，第一批替换对象是 prompts/role/*.md。
Q3：正式默认角色统一为 character_id=xiaoqi、display_name=陆小七、assistant sender_name=小七。
Q4：模型层接入 xiaoqi_prompt_package_v0.3.1，通过 PromptRegistry 加载，不直接加载 legacy reference 到普通聊天主 prompt。
Q5：人格切换不得破坏现有 /chat SSE、message 字段、quote_ref P0 和 Android 兼容字段。
Q6：关系数值、防御值、hidden_mood、prompt、verifier_report 等内部字段不得返回 UI。
```

## 2026-04-27 前端关于页文案

```text
Q1：「关于陆小七」使用正式陆小七口径，不再使用“暮 / 书店店员 / 城南老书店”。
Q2：「关于 Lumen」定义为单一角色生活流 AI 伴侣系统，不写成“能回答问题的助手”。
Q3：关于页避免“她是 AI · 一个温柔的陪伴”等泛化工具化表达。
Q4：文案核心应表达小七有自己的生活，用户是在慢慢靠近她，不是拥有她。
```

# Codex 接手提示词

把下面这段直接复制到 Codex 新对话里：

```text
你现在接手 Lumen / 陆小七项目的五模型协作仓库。

请先读取并遵守以下文件：

1. AGENTS.md
2. docs/05_LATEST_COLLAB_DOCS.md
3. roles/pm/memory/ROLE_MEMORY.md
4. roles/pm/memory/CURRENT_CONTEXT.md
5. roles/pm/memory/DECISIONS_CACHE.md
6. docs/adr/
7. 如果任务涉及具体主题，再读取 docs/collaboration/{topic}/

项目核心定位：
Lumen 不是普通 AI 女友聊天 App，而是一个单一精品角色的生活流 AI 伴侣系统。核心角色是陆小七。小七不是工具、客服、用户附属品，而是有自己生活、房间、日记、朋友圈、照片、情绪和私密空间的猫系少女。用户不是拥有她，而是在一点点进入她的人生。

当前工程规则：
- 每个角色的正式成果放 roles/{role}/completed_design_files/
- 每个角色的长期记忆放 roles/{role}/memory/
- 所有协作中内容统一放 docs/collaboration/{topic}/
- 正式交接放 docs/handoffs/
- 重大决策放 docs/adr/
- 不再使用各角色自己的 collaboration_files/

当前 PM 已拍板：
1. POST /chat 保持 message 字段，不改 content。
2. quote_ref P0 只支持 type=message。
3. quote_ref sender_name：assistant=小七，user=你。
4. 同 user_id 并发 /chat 返回 409 conversation_busy。
5. quote_ref 作为当前 user message prefix 注入模型上下文。
6. 本期不做 event_log / memory extractor，只存 messages.quote_ref。

你的任务开始前，先总结你从这些文件中读到的项目现状，再执行我给你的具体任务。
```

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

## 2026-04-28 小七画像参考图

```text
Q1：小七本人相关画像必须参考 `/root/companion/backend/res/xiaoqi.png`。
Q2：眼睛是固定身份特征,眼镜是可变配饰。
Q3：不得只靠文字 prompt 随机生成小七本人外观。
Q4：如工具链暂不支持 reference image,需说明限制,不得声称已实现参考图一致性。
```

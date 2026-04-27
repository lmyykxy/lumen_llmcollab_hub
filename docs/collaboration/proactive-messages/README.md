# 主动消息(proactive messages)协作区

## 用途

AI 主动发消息给用户的端到端机制 — 后端如何触发、协议、客户端接入。

```text
触发源(scheduler / 用户自定义 schedule)
SSE 推送协议 (/users/{id}/subscribe)
客户端 SSE 接入
重连 + 补拉
UI 渲染契约
```

跟 `/chat`(用户主动发消息)是**两条独立链路**,SSE 端点也不同。

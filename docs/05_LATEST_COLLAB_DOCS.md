# 最新协作文档追踪表

> 五个模型开始任何任务前，先读本文件。
> 当前结构版本：v0.6
> 本版变更：仓库收敛为最小协作结构，只保留角色记忆、角色正式文档、公共协作区和本追踪表。

---

## 1. 当前项目状态

Lumen / 陆小七是单一精品角色的生活流 AI 伴侣系统。
本仓库不是 App 源码仓库，只保存五模型协作所需的长期记忆、正式文档、公共协作记录和最新追踪。

当前优先级：

```text
P0：quote_ref 聊天消息引用前后端闭环与 Android Phase 3 UI 联调
P1：前端 6 屏 MVP 稳定
P2：后端模型层接入 PromptRegistry / 小七人格文件
P3：日记 / 相册 / 图片详情后续扩展
```

---

## 2. 保留结构

```text
docs/
  05_LATEST_COLLAB_DOCS.md
  collaboration/
    quote-ref/

roles/
  pm/
    memory/
    completed_design_files/
  design/
    memory/
    completed_design_files/
  android/
    memory/
    completed_design_files/
  backend/
    memory/
    completed_design_files/
  model/
    memory/
    completed_design_files/
```

规则：

```text
五模型长期上下文 → roles/{role}/memory/
角色自己的正式文档 → roles/{role}/completed_design_files/
跨角色协作记录 → docs/collaboration/{topic}/
最新状态与拍板入口 → docs/05_LATEST_COLLAB_DOCS.md
```

已移除：

```text
.github/
docs/adr/
docs/handoffs/
docs/shared/
roles/{role}/_archive/
根目录 README / CHANGELOG / Codex handoff / GitHub 上传说明
空占位协作主题
```

重大决策不再单独放 ADR 文件，必须沉淀到本追踪表和相关角色 memory。

---

## 3. 必读顺序

```text
1. docs/05_LATEST_COLLAB_DOCS.md
2. roles/pm/memory/ROLE_MEMORY.md
3. roles/pm/memory/CURRENT_CONTEXT.md
4. roles/pm/memory/DECISIONS_CACHE.md
5. 与任务相关的 docs/collaboration/{topic}/
6. 与任务相关的角色 memory/
```

---

## 4. 当前正式文档

| 文档 ID | 文档名 | 实际路径 | 负责人 | 状态 | 版本 | 用途 |
|---|---|---|---|---|---|---|
| DOC-BE-001 | Lumen_后端模型层实现规划_v0.1.md | `roles/pm/completed_design_files/Lumen_后端模型层实现规划_v0.1.md` | PM | active | v0.1 | 后端模型层升级规划 |
| DOC-PROMPT-001 | 陆小七完整人格提示文件夹 | `roles/pm/completed_design_files/xiaoqi_complete_prompt_package_v0_3_1_full.zip` | PM / Model | active | v0.3.1 | 小七人格、视觉、生图和后端接入提示词包 |

---

## 5. 当前公共协作

| 主题 | 路径 | 状态 | 当前结论 |
|---|---|---|---|
| quote_ref 聊天消息引用 | `docs/collaboration/quote-ref/` | active | 后端字段确认和适配已完成；Android 已完成数据层与 ViewModel，等待 Phase 3 UI 与联调 |

quote_ref 当前协作文件：

```text
docs/collaboration/quote-ref/01_安卓前端_Phase1_2_进度同步与字段确认.md
docs/collaboration/quote-ref/02_后端_字段确认回复.md
docs/collaboration/quote-ref/README.md
```

---

## 6. 当前 PM 拍板

### quote_ref

```text
1. POST /chat 保持 message 字段，不改 content。
2. quote_ref P0 只支持 type=message。
3. quote_ref sender_name snapshot：assistant=小七，user=你。
4. 同 user_id 并发 /chat 返回 409 conversation_busy。
5. quote_ref 作为当前 user message prefix 注入模型上下文。
6. P0 不做 event_log / memory extractor，只存 messages.quote_ref。
```

### quote_ref 后端确认

```text
POST /chat:
{
  "user_id": 123,
  "message": "我就笑,怎么了",
  "image_url": null,
  "quote_ref": {
    "type": "message",
    "id": "456"
  }
}

quote_ref.type = "message"
quote_ref.id 请求可接受 int 或 string
GET /messages quote_ref.id 响应为 number
错误体为扁平 {code, message}
400 invalid_quote_ref
409 conversation_busy
纯图 preview_text 后端兜底为 "[图片]"
模型上下文注入已由后端完成
```

### 小七产品边界

禁止未经 PM 拍板引入：

```text
好感度 / 亲密度数字
任务中心
多角色市场
Prompt 编辑器
底部 Tab 作为 MVP 主导航
登录页作为 MVP 正式入口
关系等级 UI
工具式助手语气
情感勒索或“你不回来我活不下去”式依赖
```

### 视觉身份

固定身份锚点：

```text
头发
帽子
眼睛
面貌气质
体型
```

可变元素：

```text
眼镜
穿着
姿势
表情强度
光照
场景
```

关键修正：

```text
眼睛是固定身份特征。
眼镜是可变配饰。
```

---

## 7. 维护规则

做任何文档或协作改动后检查：

```text
[ ] 是否更新本追踪表
[ ] 是否更新相关角色 memory
[ ] 是否放入 roles/{role}/completed_design_files/
[ ] 是否放入 docs/collaboration/{topic}/
[ ] 是否把重大 PM 拍板写回本追踪表
[ ] 是否影响 Android / Backend / Model 的当前上下文
```

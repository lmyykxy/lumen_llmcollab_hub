# Lumen / 陆小七 · 后端模型层实现规划 v0.1

> 面向对象：后端模型层大模型 / 后端工程实现会话  
> 当前阶段：在现有 Chat + SSE + generate_image 工具链基础上，升级为“角色状态 + 事件流 + 记忆 + 视觉元数据 + prompt 文件系统”的模型层。  
> 核心原则：不推翻当前已跑通的 `/chat` SSE 和图片工具链；通过兼容式加表、加字段、加编排层，逐步把系统从“聊天工具链”推进到“生活流角色大脑”。

---

## 0. PM 结论

当前后端已经解决了 Chat MVP 中最影响体验的问题：

- `/chat` SSE 可流式返回。
- `generate_image` 工具链已接入。
- `tool_call` 已前置，前端可立即显示“正在画图”。
- image message 已支持 `content + image_url` 单气泡图文。
- img2img 默认“怕错就填 reference”，更利于主体保留。

下一阶段不要立刻重做完整长期愿景，也不要推翻当前 API。应按以下路径演进：

```text
P0：稳定当前工具链
P1：补消息 v2 / 图片元数据 / quote_ref / event_log
P2：接入小七人格 prompt 文件系统
P3：建立 CharacterState / RelationshipState / Memory 的最小闭环
P4：接入 LifeTick / ProactiveMessage / Moment / Diary
P5：等视觉资产稳定后替换为 Visual Consistency Pipeline
```

最终模型层目标：

```text
用户消息
  ↓
Conversation Lock
  ↓
写入 Message + EventLog
  ↓
读取 CharacterState / RelationshipState / Memory / RecentEvents
  ↓
PromptRegistry 加载小七人格文件
  ↓
ContextBuilder 组装上下文
  ↓
ChatOrchestrator 调用主模型流式回复
  ↓
ToolExecutor 执行 generate_image / web_search / schedule_message
  ↓
PostProcessor 过滤工具元话术、修 caption、格式化 SSE
  ↓
持久化 Assistant Message / Image / Event
  ↓
TurnAnalyzer 异步抽取记忆、更新关系和状态
```

---

# 1. 当前架构判断

## 1.1 当前已有能力

当前可视为：

```text
FastAPI 后端
  ├─ POST /users
  ├─ POST /chat  流式 SSE
  ├─ GET /users/{id}/messages
  ├─ GET /users/{id}/subscribe
  ├─ POST /images/upload
  ├─ GET /uploads/...
  ├─ generate_image 工具
  ├─ schedule_message 工具
  ├─ web_search 工具
  └─ vision 预描述自动机制
```

当前消息协议为：

```json
{
  "id": 1,
  "role": "user | assistant",
  "content": "...",
  "image_url": "/uploads/...",
  "created_at": "iso8601"
}
```

当前 `content × image_url` 有四种情况：

```text
1. 纯文字
2. 单气泡图 + 文
3. 纯图
4. content 空且无图，不应出现
```

## 1.2 当前主要不足

当前系统仍然是：

```text
Message-driven Chat System
```

还不是：

```text
Character-state-driven Life System
```

缺口包括：

```text
1. message 缺 type / quote_ref / image_id / source_event_id
2. image 只是 url，没有 GeneratedImage 元数据
3. 没有 event_log，后续日记 / 朋友圈 / 回忆无来源
4. 没有 CharacterState，Hero / LifeFeed 无后端驱动
5. 没有 RelationshipState，误解 / 防御 / 修复无法稳定
6. 没有 Memory 系统，只靠聊天上下文
7. Prompt 没有版本化文件系统，难以持续迭代小七人格
8. generate_image 仍偏自由生图，还不是受控 ImageIntent 管线
9. 同 user_id 并发保护不足
10. 无鉴权，短期可接受，但要标注边界
```

---

# 2. 目标模型层架构

## 2.1 分层设计

```text
API Layer
  ├─ /chat
  ├─ /messages
  ├─ /character_state
  ├─ /moments
  ├─ /diary
  └─ /images

Orchestration Layer
  ├─ ChatOrchestrator
  ├─ ToolOrchestrator
  ├─ PromptRegistry
  ├─ ContextBuilder
  ├─ ResponsePostProcessor
  └─ TurnAnalyzer

Character Brain Layer
  ├─ CharacterStateService
  ├─ RelationshipService
  ├─ MemoryService
  ├─ LifeTickService
  ├─ DiaryService
  ├─ MomentService
  └─ ProactiveMessageService

Visual Layer
  ├─ ImageIntentBuilder
  ├─ ImagePromptBuilder
  ├─ ImageToolExecutor
  ├─ ImageMetadataExtractor
  └─ VisualVerifier，后期

Persistence Layer
  ├─ messages
  ├─ generated_images
  ├─ event_log
  ├─ character_states
  ├─ relationship_states
  ├─ memories
  ├─ moments
  ├─ diary_entries
  ├─ prompt_versions
  └─ model_run_logs
```

---

# 3. Prompt 文件系统接入方案

## 3.1 文件目录建议

把小七人格文件包放入：

```text
prompts/
  characters/
    xiaoqi/
      v0.3.1/
        00_README.md
        01_CHARACTER_BIBLE.md
        02_SOUL_EXPANDED.md
        03_VOICE_AND_LANGUAGE.md
        04_EMOTIONAL_ARCHITECTURE.md
        05_RELATIONSHIP_ARCS.md
        06_LIFEFLOW_ENGINE.md
        07_MEMORY_SYSTEM.md
        08_DIARY_MOMENTS_PHOTO.md
        09_VISUAL_BIBLE.md
        10_SCENE_LIBRARY.md
        11_IMAGE_PROMPT_GUIDE.md
        12_SAFETY_AND_OUTPUT.md
        13_SYSTEM_PROMPT_TEMPLATE.md
        visual_patch_v0.3.1.md
```

其中 `v0.3.1` 是当前建议加载版本。

## 3.2 PromptRegistry

新增 `PromptRegistry`，负责：

```text
1. 启动时读取 prompt 文件
2. 计算 sha256
3. 缓存到内存
4. 生成短版 prompt cards
5. 记录当前 prompt_version
6. 支持热更新，非 MVP 必须
```

建议结构：

```python
@dataclass
class PromptBundle:
    character_id: str
    version: str
    files: dict[str, str]
    sha256: str
    core_card: str
    voice_card: str
    safety_card: str
    visual_card: str
```

## 3.3 不要每次把完整文件全塞进模型

为了高效，prompt 文件应分成三层：

```text
A. Core Cards，每次聊天加载
B. Task Cards，按任务加载
C. Full Docs，仅用于离线总结 / 生成日记 / 图像计划 / 回归测试
```

### 每次聊天加载

```text
core_card              约 600-900 tokens
voice_card             约 300-500 tokens
safety_card            约 200-400 tokens
relationship_policy    约 300-500 tokens
current_state          动态 JSON
retrieved_memories     top-k
recent_events          最近 N 条
recent_messages        最近 N 条
```

### 图片生成加载

```text
visual_card
image_rules_card
current_state
source_event
reference image metadata
```

### 日记 / 朋友圈生成加载

```text
identity summary
soul summary
content_rules
life_events
memory_summary
relationship summary
```

## 3.4 Prompt 版本必须入库

每次模型运行都记录：

```text
model_run_logs.prompt_version
model_run_logs.prompt_sha256
messages.prompt_version
messages.model_name
generated_images.prompt_version
```

这样后面换 prompt 或模型时能回归测试。

---

# 4. ChatOrchestrator 设计

## 4.1 主流程

```python
async def stream_chat(user_id: str, req: ChatRequest):
    async with conversation_lock(user_id):
        user_msg = message_repo.create_user_message(req)
        event_repo.append(type="user_message", source_message_id=user_msg.id)

        state = character_state_service.get_or_create(user_id)
        relation = relationship_service.get_or_create(user_id)
        recent_events = event_repo.list_recent(user_id, limit=20)
        memories = memory_service.retrieve(user_id, query=req.text, top_k=8)
        recent_messages = message_repo.list_recent(user_id, limit=20)

        prompt = context_builder.build_chat_prompt(
            prompt_bundle=prompt_registry.get("xiaoqi", "v0.3.1"),
            user_message=req,
            state=state,
            relation=relation,
            recent_events=recent_events,
            memories=memories,
            recent_messages=recent_messages,
        )

        async for event in llm_stream_with_tools(prompt):
            yield sse_event(event)

        assistant_messages = persist_outputs(...)
        enqueue_turn_analysis(user_id, user_msg, assistant_messages)
```

## 4.2 为什么 TurnAnalyzer 异步做

聊天流式回复要快，不能等记忆抽取、关系更新、日记种子生成。

因此：

```text
同步：生成回复、工具调用、消息入库、SSE 返回
异步：记忆抽取、关系更新、状态更新、event 衍生、图片元数据补全
```

这样能降低用户感知延迟。

---

# 5. 数据库联合设计

## 5.1 保持兼容的 messages v2

当前不破坏旧协议，新增列：

```sql
ALTER TABLE messages ADD COLUMN message_type TEXT DEFAULT 'text';
ALTER TABLE messages ADD COLUMN image_id UUID NULL;
ALTER TABLE messages ADD COLUMN quote_type TEXT NULL;
ALTER TABLE messages ADD COLUMN quote_id TEXT NULL;
ALTER TABLE messages ADD COLUMN quote_preview TEXT NULL;
ALTER TABLE messages ADD COLUMN quote_thumbnail_url TEXT NULL;
ALTER TABLE messages ADD COLUMN source_event_id UUID NULL;
ALTER TABLE messages ADD COLUMN prompt_version TEXT NULL;
ALTER TABLE messages ADD COLUMN model_name TEXT NULL;
```

API 暂时仍可返回旧字段，同时新增字段：

```json
{
  "id": 123,
  "role": "assistant",
  "type": "image",
  "content": "别放大看。",
  "image_url": "/uploads/...",
  "image_id": "img_001",
  "quote_ref": null,
  "source_event_id": "evt_001",
  "created_at": "iso8601"
}
```

## 5.2 generated_images

```sql
CREATE TABLE generated_images (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT NOT NULL DEFAULT 'xiaoqi',
  message_id BIGINT NULL,
  source_event_id UUID NULL,
  source_type TEXT NOT NULL, -- chat / moment / diary / album / private
  image_url TEXT NOT NULL,
  caption TEXT NULL,
  reference_image_url TEXT NULL,
  prompt_summary TEXT NULL,
  image_intent JSONB NULL,
  atmosphere_tags TEXT[] DEFAULT '{}',
  visible_objects TEXT[] DEFAULT '{}',
  visual_version TEXT NULL,
  outfit_id TEXT NULL,
  room_id TEXT NULL,
  outside_view_id TEXT NULL,
  verifier_report JSONB NULL,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

UI 只显示：

```text
image_url
caption
atmosphere_tags
created_at
source_type
```

不要给 UI 暴露：

```text
prompt_summary
verifier_report
outfit_id
room_id
outside_view_id
```

## 5.3 event_log

最小事件表：

```sql
CREATE TABLE event_log (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT NOT NULL DEFAULT 'xiaoqi',
  type TEXT NOT NULL,
  summary TEXT NOT NULL,
  payload JSONB DEFAULT '{}',
  visibility TEXT NOT NULL DEFAULT 'internal',
  source_message_id BIGINT NULL,
  source_image_id UUID NULL,
  importance FLOAT DEFAULT 0.5,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

第一批事件类型：

```text
user_message
assistant_message
image_generated
image_uploaded
tool_called
tool_failed
memory_created
relationship_updated
character_state_updated
proactive_message_sent
moment_created
diary_created
```

## 5.4 character_states

```sql
CREATE TABLE character_states (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT NOT NULL DEFAULT 'xiaoqi',
  state JSONB NOT NULL,
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);
```

`state` 示例：

```json
{
  "location": "bedroom",
  "activity": "刚画完一张图",
  "public_mood": "懒懒的",
  "hidden_mood": "有点想被看见",
  "availability": "can_reply_slowly",
  "weather": "小雨",
  "outfit_id": "home_lazy_v1",
  "room_state": {
    "cleanliness": 0.42,
    "visible_objects": ["画稿", "手机", "布丁杯"]
  },
  "status_line": "窗外小雨，她回得会慢一点"
}
```

## 5.5 relationship_states

```sql
CREATE TABLE relationship_states (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT NOT NULL DEFAULT 'xiaoqi',
  trust FLOAT DEFAULT 0.3,
  intimacy FLOAT DEFAULT 0.2,
  security FLOAT DEFAULT 0.3,
  curiosity FLOAT DEFAULT 0.4,
  dependence FLOAT DEFAULT 0.1,
  distance FLOAT DEFAULT 0.5,
  misunderstanding FLOAT DEFAULT 0.0,
  defense_level FLOAT DEFAULT 0.2,
  willingness_to_share FLOAT DEFAULT 0.2,
  stage TEXT DEFAULT 'known',
  summary TEXT DEFAULT '',
  updated_at TIMESTAMP NOT NULL DEFAULT now()
);
```

这些数值永远不直接返回 UI。

## 5.6 memories

如果已有 pgvector：

```sql
CREATE TABLE memories (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT NOT NULL DEFAULT 'xiaoqi',
  memory_type TEXT NOT NULL,
  content TEXT NOT NULL,
  visibility TEXT NOT NULL DEFAULT 'internal',
  emotional_tags TEXT[] DEFAULT '{}',
  importance FLOAT DEFAULT 0.5,
  confidence FLOAT DEFAULT 0.8,
  source_event_id UUID NULL,
  embedding VECTOR(1536),
  created_at TIMESTAMP NOT NULL DEFAULT now(),
  last_used_at TIMESTAMP NULL
);
```

记忆类型：

```text
user_fact
relationship_event
emotional_memory
private_memory
misunderstanding
shared_joke
visual_preference
```

## 5.7 model_run_logs

```sql
CREATE TABLE model_run_logs (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id TEXT DEFAULT 'xiaoqi',
  task_type TEXT NOT NULL,
  model_name TEXT NOT NULL,
  prompt_version TEXT,
  prompt_sha256 TEXT,
  input_tokens INT,
  output_tokens INT,
  latency_ms INT,
  success BOOLEAN,
  error_message TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

用于回归、成本统计和问题排查。

---

# 6. 上下文构建策略

## 6.1 ContextBuilder 输入

```python
@dataclass
class ChatContext:
    prompt_bundle: PromptBundle
    user_message: ChatRequest
    character_state: CharacterState
    relationship_state: RelationshipState
    recent_events: list[Event]
    retrieved_memories: list[Memory]
    recent_messages: list[Message]
    quote_ref: QuoteRef | None
```

## 6.2 上下文预算

建议预算：

```text
Core identity / soul / voice:        1200-1800 tokens
Safety and output boundaries:         300-500 tokens
CharacterState JSON:                  200-400 tokens
Relationship summary:                 200-400 tokens
Retrieved memories:                   800-1500 tokens
Recent messages:                     1000-2500 tokens
Recent events:                        500-1000 tokens
Tool instructions:                    500-800 tokens
```

总量尽量控制在 6k-9k tokens 内。

## 6.3 记忆检索排序

推荐综合排序：

```text
score = semantic_similarity * 0.45
      + recency_score * 0.20
      + importance * 0.25
      + unresolved_bonus * 0.10
```

其中 unresolved_bonus 用于：

```text
未修复误解
近期情绪事件
用户刚提到的旧话题
```

## 6.4 最近消息压缩

不要无限塞历史。

策略：

```text
最近 10-20 条原文
更早内容靠 conversation_summary
关系重大事件靠 memories
```

可加表：

```sql
conversation_summaries
- user_id
- character_id
- summary
- covered_message_id_until
- updated_at
```

---

# 7. 模型调用分工

## 7.1 主聊天模型

职责：

```text
自然对话
工具调用决策
caption 生成
情绪表达
嘴硬 / 沉默 / 转移话题
```

要求：流式。

## 7.2 TurnAnalyzer 小模型

职责：

```text
记忆候选抽取
关系变化判断
事件摘要
防御状态更新建议
日记 seed
朋友圈 seed
```

不需要流式，低成本模型即可。

输入：本轮 user message + assistant output + 当前 state。  
输出：结构化 JSON。

## 7.3 ImageIntentBuilder

职责：

```text
把聊天上下文 / 朋友圈 / 日记事件转成 ImageIntent
```

不是直接 prompt，而是结构化意图。

## 7.4 ImagePromptBuilder

职责：

```text
ImageIntent + VISUAL_BIBLE → 生图 prompt / negative prompt
```

## 7.5 VisualVerifier，后期

职责：

```text
校验是否还是小七
校验眼睛、头发、帽子、体型
校验眼镜是否按 must_show 出现
校验房间和窗外
校验禁止元素
```

视觉资产没稳定前先不强制做。

---

# 8. 工具调用与 SSE 改进

## 8.1 保留当前 SSE 事件

继续支持：

```text
delta
bubble_end
tool_call
tool_result
image
error
done
```

## 8.2 image 事件向后兼容扩展

当前：

```json
{
  "id": 123,
  "url": "/uploads/...",
  "caption": "随手拍的。"
}
```

建议扩展：

```json
{
  "id": 123,
  "image_id": "img_001",
  "url": "/uploads/...",
  "caption": "随手拍的。",
  "atmosphere_tags": ["卧室", "雨天"],
  "source_event_id": "evt_001"
}
```

前端旧版忽略新增字段即可。

## 8.3 工具后元话术过滤

必须在后端硬过滤：

```text
图片已生成
图在上面
已发送图片
我已经把图片发给你了
这是我生成的图片
```

不要只靠 prompt。

## 8.4 generate_image 失败处理

建议：

```text
自动重试 1 次
仍失败 → 自然失败文案
记录 tool_failed event
```

失败文案：

```text
这张没画出来。
我等会儿再试一下。
```

不要返回技术错误给小七自然回复。

---

# 9. 并发与一致性

## 9.1 user_id 会话锁

当前必须加服务端锁。

推荐：

```text
同一 user_id 同时只能有一个 /chat 流在跑
第二个请求返回 409 conversation_busy
```

响应：

```json
{
  "error": "conversation_busy",
  "message": "previous chat stream is still running"
}
```

前端可显示：

```text
小七还在想刚才那句。
```

## 9.2 client_message_id 幂等

`POST /chat` 增加可选：

```json
{
  "client_message_id": "uuid"
}
```

避免网络重试导致重复消息。

## 9.3 tool_call 日志

所有工具调用入库：

```sql
tool_call_logs
- id
- user_id
- name
- args_json
- result_json
- status
- latency_ms
- source_message_id
- created_at
```

---

# 10. CharacterState 最小闭环

## 10.1 API

```http
GET /users/{user_id}/character_state
```

返回：

```json
{
  "character_id": "xiaoqi",
  "display_name": "陆小七",
  "activity": "刚画完一张图",
  "location": "房间",
  "status_line": "窗外小雨，她回得会慢一点",
  "weather": "小雨",
  "updated_at": "iso8601"
}
```

## 10.2 来源

CharacterState 来源优先级：

```text
1. LifeTick 生成的当前状态
2. 最近事件推导
3. 用户对话引起的状态变化
4. 默认时间段模板
```

## 10.3 状态更新

TurnAnalyzer 可输出：

```json
{
  "state_delta": {
    "public_mood": "有点开心但嘴硬",
    "hidden_mood": "被用户记得小事后变软",
    "status_line": "她把那副眼镜放回桌上了"
  }
}
```

由 CharacterStateService 校验后更新，不要让模型直接全量覆盖。

---

# 11. 记忆系统实现

## 11.1 TurnAnalyzer 输出

```json
{
  "memory_candidates": [
    {
      "type": "relationship_event",
      "content": "用户夸了小七戴眼镜的照片，小七嘴硬但开心",
      "importance": 0.72,
      "confidence": 0.9,
      "visibility": "internal"
    }
  ],
  "relationship_delta": {
    "trust": 0.01,
    "intimacy": 0.02,
    "security": 0.01,
    "defense_level": -0.01
  },
  "diary_seed": "他说那张图还行，我装作没听见。",
  "moment_seed": null
}
```

## 11.2 入库前过滤

过滤规则：

```text
普通寒暄不记
短期临时信息不记
敏感隐私不记，除非用户明确要求
重复记忆合并
低 confidence 暂存或丢弃
```

## 11.3 记忆合并

如果新记忆和旧记忆相似：

```text
更新 last_confirmed_at
提高 confidence
追加 source_event_id
不重复新增
```

---

# 12. 日记与朋友圈后端路线

## 12.1 Moment 最小表

```sql
CREATE TABLE moments (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT DEFAULT 'xiaoqi',
  text TEXT NULL,
  image_id UUID NULL,
  visibility TEXT DEFAULT 'PUBLIC',
  hidden_meaning TEXT NULL,
  atmosphere_tags TEXT[] DEFAULT '{}',
  related_event_ids UUID[] DEFAULT '{}',
  expires_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT now()
);
```

## 12.2 Diary 最小表

```sql
CREATE TABLE diary_entries (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  character_id TEXT DEFAULT 'xiaoqi',
  date DATE NOT NULL,
  title TEXT,
  public_content TEXT NULL,
  locked_content TEXT NULL,
  crossed_out_content TEXT NULL,
  visibility TEXT DEFAULT 'PUBLIC',
  unlock_condition JSONB DEFAULT '{}',
  emotional_tags TEXT[] DEFAULT '{}',
  related_event_ids UUID[] DEFAULT '{}',
  created_at TIMESTAMP DEFAULT now()
);
```

## 12.3 不急着做 UI，但先做生成种子

当前阶段可以先做：

```text
TurnAnalyzer 产出 diary_seed / moment_seed
event_log 记录
暂不自动展示
```

等 UI 日记屏上线后再打开 API。

---

# 13. 图片一致性路线

## 13.1 现阶段：自由生图 + 视觉提示补丁

短期做：

```text
将 v0.3.1 视觉规则加入 ImagePromptBuilder
固定眼睛、头发、帽子、面貌气质、体型
眼镜作为可选配饰
穿着作为可变项
```

## 13.2 中期：ImageIntent + metadata

```text
聊天想画图
  ↓
ImageIntentBuilder
  ↓
ImagePromptBuilder
  ↓
generate_image
  ↓
generated_images 入库
```

## 13.3 后期：视觉一致性六件套

等小七正式视觉资产稳定后启动：

```text
1. Character Visual Bible
2. Outfit Catalog
3. Room Scene Graph
4. Outside View Catalog
5. Visual Verifier
6. Inpainting / Regeneration Pipeline
```

不要现在强行做，避免和当前自由生图链路冲突。

---

# 14. API 规划

## 14.1 现有 API 保持

```http
POST /users
POST /chat
GET /users/{id}/messages
GET /users/{id}/subscribe
POST /images/upload
GET /uploads/{id}/{file}
```

## 14.2 近期新增

```http
GET /users/{id}/character_state
GET /users/{id}/images/{image_id}
POST /users/{id}/events/view_moment
POST /users/{id}/events/view_image
```

## 14.3 中期新增

```http
GET /users/{id}/moments
GET /users/{id}/diary
GET /users/{id}/diary/{diary_id}
GET /users/{id}/album
```

## 14.4 ChatRequest v2

```json
{
  "user_id": "...",
  "content": "...",
  "client_message_id": "uuid",
  "quote_ref": {
    "type": "message | image | moment | diary",
    "id": "..."
  },
  "attachments": [
    {
      "type": "image",
      "upload_id": "..."
    }
  ]
}
```

---

# 15. 实施计划

## Phase A：当前链路收尾，1-2 天

```text
1. 工具后元话术硬过滤
2. gpt-image-1 500 自动重试 1 次
3. user_id 会话锁
4. client_message_id 幂等，若工期允许
```

验收：

```text
不会出现“图片已生成/图在上面”
同 user_id 并发不会消息交错
图片失败体验自然
```

## Phase B：数据骨架，2-4 天

```text
1. messages v2 兼容加字段
2. generated_images 表
3. event_log 表
4. quote_ref 字段
5. model_run_logs 表
```

验收：

```text
旧前端不坏
新字段可返回
图片有 image_id
事件有 source_event_id
```

## Phase C：PromptRegistry + ContextBuilder，3-5 天

```text
1. prompt 文件目录接入
2. PromptRegistry 启动加载
3. 生成 core_card / voice_card / visual_card
4. Chat prompt 改为从 prompt bundle 组装
5. 每次模型运行记录 prompt_version
```

验收：

```text
回复风格明显更像小七
不会输出旧 OpenClaw 控制标记
prompt 版本可追踪
```

## Phase D：TurnAnalyzer + Memory，5-7 天

```text
1. TurnAnalyzer 小模型结构化输出
2. memories 表 + embedding
3. relationship_states 表
4. character_states 表
5. 异步状态更新
```

验收：

```text
用户夸照片会形成记忆
用户消失会影响状态
小七能自然引用旧事
```

## Phase E：CharacterState API + ProactiveMessage，3-5 天

```text
1. GET /character_state
2. absence_tick
3. proactive message 入库
4. subscribe 推送可复用
```

验收：

```text
Hero 不再全靠假数据
久未上线有合理主动消息
状态行可被后端驱动
```

## Phase F：Moment / Diary 种子，后续

```text
1. moment_seed / diary_seed 入库
2. 最小 moments 表
3. 最小 diary_entries 表
4. UI 就绪后开放 API
```

---

# 16. 回归测试清单

## 16.1 Chat

```text
普通聊天可流式
图片生成 tool_call 立即推
图片 caption 不重复
图片消息为单气泡图文
caption 为空时前端不出空白气泡
```

## 16.2 Prompt

```text
不出现 [SENDIMG]
不出现 [MEMORY]
不出现 NO_REPLY
不每句强行喵
不把用户叫主人
不暴露系统工具
```

## 16.3 关系

```text
用户夸她 → 回复嘴硬但开心
用户消失回来 → 可轻微嘴硬
用户追问私密内容 → 可以回避
用户关心她 → 可以慢慢软下来
```

## 16.4 图片

```text
头发一致
帽子一致
眼睛固定
眼镜可戴可不戴
体型不漂移
穿着可变
```

## 16.5 数据

```text
messages 老接口兼容
generated_images 有记录
event_log 有记录
model_run_logs 有记录
同 user_id 并发被锁
```

---

# 17. 后端模型层可复制执行 Prompt

```text
你是 Lumen / 陆小七项目的后端模型层实现助手。
当前系统已有 FastAPI 后端、POST /chat SSE、GET /messages、POST /images/upload、generate_image 工具和 schedule_message 工具。不要推翻当前已跑通的 Chat 和 SSE 协议。

你的任务是把当前“聊天 + 生图工具链”升级为“角色状态 + 事件流 + 记忆 + prompt 文件系统”的模型层。

优先级：
P0：工具后元话术过滤、图片失败重试、同 user_id 会话锁。
P1：messages v2 兼容字段、generated_images、event_log、quote_ref、model_run_logs。
P2：PromptRegistry 接入 prompts/characters/xiaoqi/v0.3.1，把小七人格文件拆成 core_card / voice_card / safety_card / visual_card，通过 ContextBuilder 组装。
P3：TurnAnalyzer 异步抽取 memory_candidates、relationship_delta、diary_seed、moment_seed，写入 memories、relationship_states、character_states。
P4：新增 GET /users/{id}/character_state，后续接入 proactive message、moments、diary。

注意：
1. 不要让自然回复暴露工具名、文件路径、prompt、系统状态。
2. 不要输出旧 OpenClaw 标记：[SENDIMG]、[MEMORY]、[DAILY_PLAN]、NO_REPLY。
3. 小七不是客服，不是工具，不要每句强行“喵”。
4. 眼睛是固定身份特征，眼镜是可变配饰。
5. 关系数值、防御值、verifier_score 等内部字段不返回 UI。
6. 所有新增字段必须向后兼容现有 Android 客户端。
```

---

# 18. 最终目标

当前阶段不是一次性做完 Lumen 的全部愿景，而是建立正确的模型层骨架。

最终后端应该从：

```text
用户发消息 → LLM 回复 → 可选生图
```

升级为：

```text
用户进入小七的生活
  ↓
系统知道她此刻在哪里、在做什么、心情如何
  ↓
她基于关系、记忆、生活状态和私密心理回应
  ↓
聊天、日记、朋友圈、照片互相留下痕迹
  ↓
用户不是在用工具，而是在进入一个持续生活的人
```

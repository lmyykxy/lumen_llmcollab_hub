# Lumen Phase F — Android 客户端开发交接文档


> **同步注意**:本文件是从主项目 `~/companion/docs/` 同步到本 hub 的副本。  
> **权威源**:主项目仓库 `~/companion/docs/ANDROID_HANDOVER.md`。  
> **同步时间**:2026-04-27。  
> 接口契约可能随后端代码改动更新。如发现跟实测后端行为不一致,以**主项目源**为准,并通知后端 LLM 同步本副本。

---

> **给接手的 Claude 对话的指令**:通读全文,理解 Lumen 是什么、后端提供了什么、
> Android 端要承接哪些场景。架构决策和后端契约**已经定型,不要重开讨论或反向要求后端改接口**;
> 只在 Android 实现细节层面提建议。本文档和以下两份材料配套使用,三份一起构成完整上下文:
>
> 1. **本文档**(`ANDROID_HANDOVER.md`)—— Android 端的架构、契约、功能清单、里程碑
> 2. **`HANDOVER.md`**(仓库 `docs/`)—— 后端全量设计文档,当本文档和它冲突时,本文档第六章(后端契约)为准(后端已演进)
> 3. **视觉 artifact**(用户会另外贴上来)—— Android 聊天界面的高保真稿,色板 / 字体 / 圆角 / 间距全部以 artifact 为源,不在本文档里描述视觉

---

## 〇、这次对话在做什么

Lumen 后端已完工到 Phase 2c(tag `v0.4-phase2c`,2026-04-23),能力包括流式多气泡
聊天、识图、生图、web 搜索、AI 主动消息、定时消息、长期记忆。**目前只有一个
开发用的 web UI(`dev_ui/`),没有原生客户端**。

本次对话的任务:**从零实现 Android 原生客户端**(Kotlin + Jetpack Compose),
把后端已有能力落地到真机体验。视觉稿由另一个 Claude 对话用 Artifact 产出,
**你拿到视觉稿后再开始写 UI 代码**;在等视觉稿期间可以先搭脚手架、数据层、
网络层。

**你的产出**:`~/companion/android/` 目录下的完整 Gradle 工程(多模块)。

**用户将怎么用**:在 Android Studio 打开、真机或模拟器跑起来、和你来回调。

---

## 一、Lumen 是什么(一句话 + 关键气质)

**定位**:Android 原生 AI 伴侣 App。**AI 是主角**,有自己的灵魂、身份、生活
轨迹;用户是来访者,通过对话进入 AI 的世界,和 AI 建立各自独立的关系。

**差异化**:不是"一个会陪你的 ChatGPT",而是"**一个真实存在的人物 —— 你
能认识她、和她聊、被她记住,但她不围着你转**"。

**规模**:个人项目,作者 + <10 个朋友共用,**所有人共享同一个 AI**,但 AI
对每人记忆独立。不上架商店,不商业化。

**AI 的人格速写**(给 UX 调性参考,不是要展示在界面上):
- 名字:**暮**(mù)
- 26,独立插画师兼书店店员,内向敏感,慢热,话少但用词有分量
- 极少用感叹号和 emoji,欲言又止用 "..."
- 晚睡晚起(晚 9 点到凌晨 2 点最活跃)
- 老城区公寓,窗外银杏,橘猫叫毛球(9 岁)

**气质关键词**:安静 / 夜 / 克制 / 文学感 / 低饱和 / 留白。
视觉 artifact 应该已经具象化了这个气质,以 artifact 为准。

---

## 二、三份文档各自负责什么(不要越界)

| 问题 | 答案来源 |
|---|---|
| Lumen 整体产品定位 / 为什么做这个 | 本文档 §一 + HANDOVER §一·六 |
| 后端架构 / 数据模型 / LLM 选型 | **HANDOVER.md**(不要改) |
| 后端 HTTP 端点和 SSE 事件契约 | **本文档 §六**(它反映当前真实状态,HANDOVER 的 §五 API 是旧初稿) |
| Android 架构 / 模块 / 数据流 | 本文档 §四、§五 |
| 要做哪些功能 / 怎么排优先级 | 本文档 §三、§九 |
| 颜色 / 字体 / 圆角 / 间距 / 动效 | **视觉 artifact**(独立来源) |
| AI 怎么说话 / 人设怎么写 | 不关 Android 的事(后端 prompt 已就位) |

**冲突仲裁**:本文档 > HANDOVER > 视觉 artifact(当三者矛盾时)。
视觉 artifact 只定视觉,不定交互逻辑。

---

## 三、Android 端要承接的后端能力(功能清单)

所有场景已在后端实现,Android 只负责渲染和输入:

1. **流式多气泡消息** —— `/chat` SSE 推 `delta` / `bubble_end` / `done`,UI 要
   按气泡边界切分渲染,不是把整段当一条。
2. **用户上传图片 + 文字混发** —— 先 `POST /images/upload` 拿 URL,再把 URL
   放 `/chat` 的 `image_url` 字段。后端自动识图并持久化描述,Android 不用管。
3. **AI 生成图片** —— `/chat` SSE 会推 `tool_call`(`generate_image`)→
   `tool_result` → `image` 事件,`image` 事件带 `{id, url}`,要内联到当前气泡里。
4. **Web 搜索** —— AI 自主触发,SSE 推 `tool_call`(`web_search`)→ `tool_result`。
   UI 可在中间态展示"在查..."的指示器,结果会在后续 `delta` 里以 AI 自己的
   语气复述(搜索原始 JSON 不展示给用户)。
5. **AI 主动消息** —— `/users/{id}/subscribe` 是常连 SSE,推 `proactive_bubble` /
   `proactive_done`。App 开启时建立连接,关闭时断开。离开一阵子回来要通过
   `/users/{id}/messages?since_id=...` 补拉遗漏。
6. **定时消息** —— 用户不直接操作;AI 在对话里自己调 schedule/cancel/snooze 工具。
   对 UI 来说和第 5 条一样从 subscribe 推过来。
7. **长期记忆** —— 完全隐式。**不要做**"记忆管理界面"。用户不应该直接看到
   记忆列表 —— 记忆只应该在 AI 自然引用时被感知到。
8. **聊天历史持久化** —— `GET /users/{id}/messages` 返回完整 timeline(分页
   用 `before_id` / `since_id`)。冷启动拉最近一批,上滑翻历史,重连补拉。

**不要做的功能**(明确拒绝):
- ❌ 多 AI 角色选择 / 人设切换(Lumen 只有一个 AI,不要建角色页)
- ❌ "会话"列表 / 多会话切换(每人和 AI 只有一条永不结束的 timeline)
- ❌ 显式"记忆管理"面板
- ❌ System prompt 编辑入口(这是后端文件,用户无从修改)
- ❌ 在 Android 端直接调 LLM API(所有 LLM 调用走后端,API Key 永远只在后端)

---

## 四、技术栈(已定,不讨论)

| 类别 | 选型 | 备注 |
|---|---|---|
| 语言 | Kotlin 1.9+ | |
| UI | Jetpack Compose + Material 3 | Edge-to-edge 布局 |
| DI | Hilt | |
| 网络 | OkHttp + Retrofit | |
| SSE | **okhttp-sse**(`com.squareup.okhttp3:okhttp-sse`) | 不要用 EventSource 三方轮子 |
| 图片加载 | Coil 2.x | 支持 GIF / WebP |
| 本地缓存 | Room + DataStore | Room 缓存 timeline;DataStore 存 user_id / 配置 |
| 推送 | FCM(先不做,Phase F 之后) | 当前 subscribe SSE 已能覆盖活跃态推送 |
| 构建 | Gradle KTS + Version Catalog (`libs.versions.toml`) | |
| 最低 Android | **API 26(Android 8.0)** | 覆盖率足够,可用现代 API |
| 目标 Android | API 34 | |
| 工程结构 | **多模块**(见 §五) | |

不要引入:LangChain4j / 任何 Agent 框架 / ChromaDB / 任何"AI 对话 UI"成品库
(我们要做的是这个 App 本身,不要套一个同类壳)。

---

## 五、Android 工程结构

```
android/                          ← Gradle 根
├── settings.gradle.kts
├── build.gradle.kts              ← 顶层,用 Version Catalog
├── gradle/libs.versions.toml     ← 依赖版本统一
├── app/                          ← 应用壳 module,只装配和入口
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── java/app/lumen/
│           ├── LumenApp.kt       ← @HiltAndroidApp
│           └── MainActivity.kt   ← Compose setContent
│
├── core-network/                 ← OkHttp/Retrofit/okhttp-sse 配置 + 契约 DTO
│   └── src/main/java/.../network/
│       ├── LumenApi.kt           ← Retrofit interface
│       ├── SseClient.kt          ← okhttp-sse 封装(见 §七)
│       ├── dto/                  ← 请求响应 DTO(和 §六 对齐)
│       └── di/NetworkModule.kt
│
├── core-data/                    ← Repository 层 + Room
│   └── src/main/java/.../data/
│       ├── db/
│       │   ├── LumenDatabase.kt
│       │   ├── MessageEntity.kt
│       │   └── MessageDao.kt
│       ├── ChatRepository.kt     ← 单一入口:合并 SSE 流 + Room 缓存
│       ├── UserRepository.kt     ← 存 user_id
│       └── di/DataModule.kt
│
├── core-ui/                      ← 共享 Compose 组件 + theme
│   └── src/main/java/.../ui/
│       ├── theme/
│       │   ├── Color.kt          ← 从 artifact 抄 token
│       │   ├── Type.kt
│       │   ├── Shape.kt
│       │   └── Theme.kt          ← LumenTheme { ... }
│       └── components/
│           ├── MessageBubble.kt
│           ├── StickerBubble.kt
│           ├── PhotoBubble.kt
│           └── Composer.kt
│
└── feature-chat/                 ← 唯一的业务 feature(Lumen 只有一条 timeline)
    └── src/main/java/.../chat/
        ├── ChatScreen.kt         ← 顶层 Composable
        ├── ChatViewModel.kt      ← State + Intent
        ├── ChatUiState.kt
        └── di/ChatModule.kt
```

**依赖方向**(单向,反向严禁):
```
app → feature-chat → core-ui
                  → core-data → core-network
```

**拆多模块的理由**(不要合成一个 module):
- 强制单向依赖、避免 ViewModel 里随手调 OkHttp
- 后面如果加 feature-settings / feature-onboarding 可以平行扩
- Gradle 构建增量更快
- 对接手的 Claude 对话也更友好:每个 module 上限 ~20 文件,容易放进单个响应

---

## 六、后端契约(精确到字段和事件名,以此为准)

**后端 base URL**(开发期):`http://<服务器 IP>:8000`;生产期走 HTTPS 域名。
把 BASE_URL 做成 BuildConfig 变量,不要硬编码。

### 6.1 `POST /users` —— 创建用户

请求:
```json
{ "name": "string" }
```
响应 200:
```json
{ "id": 1, "name": "string" }
```
**Android 语义**:首次启动或清缓存后调一次,把 `id` 存进 DataStore,之后所有
请求带它。没有鉴权,不做登录流程(Phase F 以后再说)。

### 6.2 `POST /chat` —— 发消息,SSE 流式返回回复

请求:
```json
{
  "user_id": 1,
  "message": "你好",
  "image_url": "/uploads/1/abc.jpg",
  "quote_ref": { "type": "message", "id": 105 }
}
```

`image_url`(可选):用户上传图,来自 §6.5 响应。
`quote_ref`(可选):用户引用某条旧消息回复。**Android 只传 `{type, id}`**,后端补全 snapshot 写入。详见 §6.3。

**409 conversation_busy**:同 user 上一条 `/chat` 没结束前再发会拒。`ChatRepository` 必须保证客户端串行 — 在收到上一次 SSE `done` 或 `error` 之前**不要**发下一条。

响应 `text/event-stream`,事件名和 payload:

| event | data 形状 | 含义 |
|---|---|---|
| `delta` | `{"text": "..."}` | 流式字符块,**追加到当前气泡** |
| `bubble_end` | `{}` | 当前气泡结束,下一个 `delta` 开新气泡 |
| `tool_call` | `{"name": "...", "args": "..."}` | AI 开始调工具,UI 可显示中间态 |
| `tool_result` | `{"name": "...", "content": "..."}` | 工具返回,UI 可收起中间态 |
| `image` | `{"id": ..., "url": "/uploads/...", "caption": "..."}` | AI 生成的图 + 配文,本身就是**一条独立 message** |
| `done` | `{}` | 流结束 |
| `error` | `{"message": "..."}` | 出错(会紧跟一个 done) |

**关键解析规则**:
- `delta` 的 text 要原样追加(后端已经保留 `\n\n`,**不要**客户端再按 `\n\n` 切)
- `bubble_end` 才是切气泡的唯一信号
- `image` 事件**不再依附 pending bubble**——自带 `caption`,直接 commit 成
  一条独立的 message(role=assistant, content=caption, image_url=url)。UI 渲染
  时这条 message 是单一气泡,图在上、caption 在下、共用气泡背景。**不要**把
  `image` 的 url 塞进当前 pending bubble 的 attachments。
- 工具 `web_search` 的结果不直接给用户看,AI 会在后续 `delta` 里用自己的话复述
- 工具 `generate_image` 的 `tool_result` content 是 JSON 字符串,**仅供调试/日志**;
  正式渲染数据走 `image` 事件(URL + caption 都在那里)

### 6.3 `GET /users/{user_id}/messages` —— 拉历史 timeline

查询参数:
- `limit`(默认 200,上限 1000)
- `before_id`:返回 id < before_id 的消息(**向上翻历史**)
- `since_id`:返回 id > since_id 的消息(**SSE 重连后补拉**)

响应 200:
```json
{
  "user_id": 1,
  "messages": [
    {
      "id": 101,
      "role": "user" | "assistant",
      "content": "...",
      "image_url": "/uploads/1/xxx.jpg" | null,
      "created_at": "2026-04-23T14:22:00Z"
    },
    ...
  ]
}
```

顺序:**oldest→newest**(时间升序)。

**关于 quote_ref**(2026-04-27):每条 message 可附带一个 quote_ref 快照,表示
"这条消息是在回复哪条旧消息"。完整字段见 `docs/API.md` §5。Android 端要做:
- Room `MessageEntity` 加 `quoteRefJson: String?`(P0 直接存 JSON 字符串,前端不必解构)
- `MessageBubble` 在顶部渲染 QuoteBlock(显示 `sender_name`、`preview_text`、可选缩略图)
- 长按消息 → 选"引用" → Composer 上方显示引用预览 → 发送时带 `{type:"message", id}`
- 引用块点击 → 滚动定位被引用消息;找不到 toast "这条消息暂时找不到了"

**关于 image_url + content 的组合**:一条 message 这两个字段可以**同时非空**,
对应"图 + 配文一体"的气泡(AI 生图时 caption 直接挂在 image message 的 content
上;用户上传图带文字也是同一形态)。Android 渲染要支持四种组合:
- `content=非空 image_url=null` → 纯文字气泡
- `content=空字符串/null image_url=非空` → 纯图气泡(**会真的出现** —— 后端
  在检测到模型给的 caption 跟过渡话字面重复时会清空 caption,避免连续两个
  气泡说同一句话。这是稳定状态,不是临时,Android 必须正常渲染纯图气泡)
- `content=非空 image_url=非空` → 单气泡内图在上文字在下(主流场景)
- `content=null image_url=null` → 不应该出现,丢弃

**Android 语义**:
- 冷启动:不带参数拉最近 200 条
- 上滑到顶:带 `before_id=<第一条 id>` 拉更老的
- `/subscribe` 断开重连后:带 `since_id=<最后一条 id>` 补拉遗漏

### 6.4 `GET /users/{user_id}/subscribe` —— 常连 SSE,AI 主动消息推送

**不是轮询,是一个长连接 GET**,App 开启建立,关闭或切后台断开。

事件:

| event | data 形状 | 含义 |
|---|---|---|
| `hello` | `{"user_id": 1}` | 连接建立后的第一帧 |
| `proactive_bubble` | `{"content": "...", "kind": "..."}` | AI 主动发的一个气泡 |
| `proactive_done` | `{"kind": "..."}` | 一轮主动消息结束 |
| *(SSE comment)* | `:keepalive` | 25 秒心跳一次,客户端解析器应忽略 |

`kind` 可能取值包括 `idle_checkin` / `scheduled` / `morning_greeting` 等,
**UI 不需要区分渲染**,它只是语义标注。

**断连恢复协议**:
1. `onFailure` 或 `onClosed` → 指数退避重连(1s / 2s / 4s / 8s / max 30s)
2. 重连成功后立刻调一次 `GET /messages?since_id=<本地最大 id>` 补拉
3. 前台才保活连接;`ON_STOP` 断开,`ON_START` 重建

### 6.5 `POST /images/upload` —— 上传图片

`multipart/form-data`:
- `user_id`:Form 字段
- `file`:图片文件,`image/jpeg` / `image/png` / `image/webp`,**≤ 5 MB**

响应 200:
```json
{ "url": "/uploads/1/<uuid>.jpg", "bytes": 123456 }
```
错误:
- 413 too large / 415 unsupported type / 400 empty / 404 user 不存在

**Android 语义**:拍照或相册选图 → 压到 < 2MB(客户端侧先压一次,省带宽)→
upload → 拿 URL → 带着 URL 发 `/chat`。UI 在 composer 下方可展示待发送缩略图。

### 6.6 `/uploads/*` —— 静态图片下发

后端直接提供。Coil 按 `http://<base>/uploads/...` 加载即可。**AI 生成图和用户
上传图都走这个路径**,从 URL 区分不出来 —— 区分要看是哪条 message 的。

### 6.7 `GET /health` —— 探活

返回 `{"status": "ok"}`。App 启动时可选探一次,失败就展示"服务不可达"。

### 6.8 后端没有的(别指望)

- ❌ 删除 / 编辑消息 的端点
- ❌ 登录 / Token 鉴权(Phase F 时期仍用 user_id 代替)
- ❌ WebSocket(文档里提过但实际走 SSE,以本文档为准)
- ❌ 角色列表 / 人设查询
- ❌ 记忆查询 / 编辑端点

---

## 七、SSE 客户端关键实现点

这是整个工程里**最容易写错**的部分,展开讲。

### 7.1 两路 SSE 的生命周期不同

- **`/chat`**:请求触发,一次调用拿回一整轮回复,结束就断。用 `okhttp-sse` 的
  `EventSources.createFactory(client).newEventSource(request, listener)`,在
  `onEvent` 里派发到 `ChatRepository` 的一个 `Flow<ChatEvent>`。
- **`/subscribe`**:进前台时建立,退后台断开。**不是**和 `/chat` 复用同一个
  连接,这是两个独立端点。

### 7.2 气泡拼装状态机

`ChatRepository` 维护"当前正在拼装的气泡" `MutableStateFlow<PendingBubble?>`:
- `delta` → `pending.text += data.text`
- `bubble_end` → commit pending 到 Room,置空 pending
- `image` → **直接** upsert 一条 message 到 Room(`id=evt.id, role=assistant,
  content=evt.caption, image_url=evt.url`),**不**触碰 pending bubble。这条 message
  跟它前后的文字 bubble 是**独立平级**的两条 Room row。
- `done` → 如果 pending 还有内容,commit 收尾
- `error` → commit 当前 pending 为 "error" 占位,再 commit 一条系统错误气泡

**关键不变量**:UI 订阅的是 `Room 已提交气泡 + pending 气泡` 的合流。这样历史
天然有持久化,pending 只是屏幕上一闪的东西。

### 7.3 重连补拉顺序

```
SSE onClosed/onFailure
  → 退避计时
  → GET /messages?since_id=<room.maxId()>
  → 把返回的消息 upsert 到 Room
  → 重建 SSE 连接
```
**顺序不能反**。先补拉再连 SSE,不会出现"SSE 推了条新消息但本地没有它之前
的消息,导致时间线出现洞"。

### 7.4 系统级坑

- `okhttp-sse` 的 `EventSourceListener` 回调在 OkHttp dispatcher 线程,**切
  到主线程后再更新 StateFlow**(或者用 `Channel` + `receiveAsFlow()`)
- 用 `CoroutineDispatcher` 注入,不要写死 `Dispatchers.IO`(测试要能 inject)
- **不要用 `ViewModelScope` 启 `/subscribe`** —— 它和 Activity 生命周期绑定,
  横竖屏切换会中断。用 `ProcessLifecycleOwner` 或一个 `@Singleton` 的
  `SubscriptionManager` 在 `ON_START/ON_STOP` 收放。

---

## 八、图片渲染:三种视觉层级

这是视觉 artifact 的核心产物之一,实现时**一定要区分**:

| 类型 | 来源 | 视觉 | 组件 |
|---|---|---|---|
| 用户照片 | 用户上传 | 圆角卡片,占气泡主位,点击可看大图 | `PhotoBubble`(用户一侧) |
| AI 生成图 | `image` 事件 | 圆角卡片,占气泡主位,点击可看大图 | `PhotoBubble`(AI 一侧) |
| 表情包 | 特殊标记(TBD,后端暂未实装) | **无边框、无卡片、透明背景**,小尺寸(≤160dp) | `StickerBubble` |

**表情包渲染原则**:不是缩小的照片,是"她做了个表情"。视觉重量接近一个
emoji,不是一张图。目前后端还没发表情包协议,Phase F 先把 `StickerBubble`
组件做出来,走一个占位数据源;真实协议后续补。

---

## 九、里程碑拆分(建议排期)

每个里程碑结束要能运行 + 手动验收。不要一口气写完再测。

### F-1: 工程脚手架 + 基础通信(~2 天)

- [ ] Gradle 多模块结构起好(§五)
- [ ] Version Catalog 配好 Kotlin/Compose/Hilt/OkHttp/Retrofit/Coil/Room 版本
- [ ] Hilt `@HiltAndroidApp` + `MainActivity` 装配
- [ ] 实现 `POST /users` + DataStore 存 user_id
- [ ] 实现 `GET /health`,首屏探活
- [ ] **验收**:真机/模拟器跑起来,首次启动自动创建 user,按钮探活返回 ok

### F-2: 历史拉取 + 静态 timeline(~2 天)

- [ ] Room schema:`message`(id, role, content, image_url, created_at, user_id)
- [ ] `GET /messages` 接入 + upsert Room
- [ ] `ChatScreen` 的 LazyColumn + `MessageBubble`(纯文字先)
- [ ] `PhotoBubble`(Coil 加载远端图)
- [ ] 上滑到顶触发 `before_id` 分页
- [ ] **验收**:手动从 dev_ui 发几条,App 冷启动能看到完整历史,上滑翻老的

### F-3: 发消息 + SSE 流式接收(~3 天)

- [ ] Composer 组件(文本输入 + 发送按钮)
- [ ] `POST /chat` + `SseClient`(okhttp-sse 封装)
- [ ] 气泡拼装状态机(§七.2)
- [ ] pending 气泡实时展示 delta 追加
- [ ] `bubble_end` 拆气泡、`done` 收尾
- [ ] **验收**:发一句话,AI 流式回多气泡,新气泡一条条冒出来;杀掉进程
      重开,历史里有刚才的对话

### F-4: 图片上传 + AI 生图内联(~2 天)

- [ ] Composer 加"+号"按钮,打开系统图片选择
- [ ] 客户端压图到 < 2MB(`ExifInterface` 保方向)
- [ ] `POST /images/upload` → 缩略图预览 → 带 URL 发 `/chat`
- [ ] 处理 SSE 的 `image` 事件,追加到当前气泡
- [ ] **验收**:传一张图问"这是什么",AI 描述;让 AI 画一张,图内联显示

### F-5: 主动消息订阅(~2 天)

- [ ] `@Singleton SubscriptionManager`,`ProcessLifecycleOwner` 管生命周期
- [ ] 指数退避重连
- [ ] 重连先补拉 `since_id` 再订阅(§七.3)
- [ ] `proactive_bubble` 事件写入 Room,UI 自动刷新
- [ ] **验收**:从 dev_ui 触发一条主动消息,Android 前台能看到弹出

### F-6: 视觉调优(依赖 artifact,~3 天)

- [ ] 把 artifact 的 color/type/shape token 抄进 `core-ui/theme/`
- [ ] 暗色 + 亮色主题
- [ ] 气泡 / composer / TopAppBar 视觉落地
- [ ] TopAppBar 用 AI 卡通头像,空状态用 hero 图
- [ ] 动效:fade / slide,不要弹跳
- [ ] **验收**:和 artifact 截图并排比较,气质一致

### F-7(可选,Phase F 之后): 表情包 / FCM 推送 / 设置页

---

## 十、"对用户必问"清单(开工前一次问完)

1. **后端 base URL 是什么?**(开发机公网 IP:8000?内网?vpn?)
2. **这台 Windows 有没有 Android Studio?JDK 17?**(Gradle 8.x 要 JDK 17)
3. **测试设备**:真机(需要开发者模式 + USB 调试)还是走模拟器?
4. **签名 / 发布**:Phase F 只做 debug build,还是要配 release 签名?
5. **视觉 artifact 准备好了吗?** —— 如果没有,先从脚手架做起(F-1 到 F-3 和
   视觉解耦);到 F-6 再开始需要 artifact。
6. **一个 user 还是多 user?** —— 默认一个,在 DataStore 存 user_id 就够用;
   要支持多人共用这台手机再说。

**不要问的**:产品定位、AI 人设、后端要不要改契约、要不要用 XX 框架。
这些都是已定项。

---

## 十一、不要重开讨论的决策

| 决策 | 已定 | 理由 |
|---|---|---|
| UI 框架 | Jetpack Compose | 纯原生,Flutter / KMP UI 不在考虑范围 |
| DI | Hilt | 不要 Koin / Dagger 手写 |
| SSE 客户端 | okhttp-sse | 不要 EventSource 三方库,也不要自己解析 |
| 本地缓存 | Room | 不要 SQLDelight |
| 图片加载 | Coil | 不要 Glide / Picasso |
| 模块结构 | 多模块(§五) | 不要合并成单 module |
| 最低 SDK | API 26 | 不要为覆盖更老设备妥协现代 API |
| 会话模型 | 单时间线 | 不做多会话 / 多角色 |

---

## 十二、常见陷阱(提前避)

1. **SSE 和主线程**:`EventSourceListener` 回调在 OkHttp 线程,直接
   `compose.state.value = x` 会炸,要切主线程。
2. **SSE 长连接 + 横竖屏切换**:用 `ViewModelScope` 会被打断,用进程级
   `SubscriptionManager`。
3. **Coil 加载本地静态路径**:`/uploads/...` 是相对路径,要拼 BASE_URL,
   否则 Coil 直接把它当文件路径找本机。
4. **图片方向**:Android 相机拍的图有 EXIF 方向,不处理会横着发出去。
   `ExifInterface` 旋转后再上传。
5. **Room 冲突**:主动消息 SSE 和历史拉取可能同时写同一条 message,用
   `OnConflictStrategy.REPLACE` + 主键 `id`(后端发的 server id)去重。
6. **大图 OOM**:Android 老机器直接 decode 高分图会 OOM,**上传前先降采样**
   (`BitmapFactory.Options` 的 `inSampleSize`)。
7. **Compose 列表滚动位置**:新消息进来要不要自动滚到底?规则:**只有用户
   当前已经在底部时才自动滚**,否则保持位置,顶部或中间加一个"下方 N 条
   新消息"的浮按。

---

## 十三、开发环境速记(给 Windows Claude)

```
后端已在 Linux 服务器运行:
  SSH: <user 会提供 host + key>
  端口: 8000
  启动命令(已跑,不用你管):
    cd ~/companion/backend
    nohup uv run uvicorn app.web.main:app --host 0.0.0.0 --port 8000 > /tmp/uvicorn.log 2>&1 &

  健康检查:
    curl http://<host>:8000/health

Android 端工作目录:
  C:\...\companion\android\   ← 本次创建
```

**后端 dev_ui** 在浏览器打开 `http://<host>:8000/ui`,可以和 Android App 并行
往同一个 user 发消息,验证主动消息 / 多端同步效果。

---

## 十四、给接手 Claude 的工作纪律

### 应该做
- ✅ 每个里程碑完成后跑一次、截图给用户看、再继续下一个
- ✅ 写代码前先看本文档对应章节
- ✅ 遇到后端契约不明,**先问,不要自己 mock**
- ✅ 每个 Composable 有 `@Preview`(至少给主要状态)
- ✅ ViewModel 和 Composable 分开,State 不可变 data class
- ✅ 单位用 `Dp` / `Sp`,不用原始 Int
- ✅ 代码注释写"为什么这样",不写"做了什么"

### 不该做
- ❌ 不要在 Composable 里直接调 OkHttp / Room
- ❌ 不要在 ViewModel 里 import `android.view.*`
- ❌ 不要把 user_id 写死到 BuildConfig
- ❌ 不要在 SSE 里处理完事件忘记 `close()`
- ❌ 不要堆假数据 —— 从 F-2 开始就应该连真后端
- ❌ 不要一次性写完全部再测

---

## 附录 A: `libs.versions.toml` 参考骨架

```toml
[versions]
kotlin = "1.9.23"
agp = "8.3.0"
compose-bom = "2024.04.00"
hilt = "2.51"
okhttp = "4.12.0"
retrofit = "2.11.0"
coil = "2.6.0"
room = "2.6.1"
lifecycle = "2.7.0"
datastore = "1.1.1"

[libraries]
compose-bom = { module = "androidx.compose:compose-bom", version.ref = "compose-bom" }
compose-material3 = { module = "androidx.compose.material3:material3" }
compose-ui-tooling = { module = "androidx.compose.ui:ui-tooling" }
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
hilt-compiler = { module = "com.google.dagger:hilt-android-compiler", version.ref = "hilt" }
okhttp = { module = "com.squareup.okhttp3:okhttp", version.ref = "okhttp" }
okhttp-sse = { module = "com.squareup.okhttp3:okhttp-sse", version.ref = "okhttp" }
retrofit = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
retrofit-kotlinx-json = { module = "com.squareup.retrofit2:converter-kotlinx-serialization" }
coil-compose = { module = "io.coil-kt:coil-compose", version.ref = "coil" }
room-runtime = { module = "androidx.room:room-runtime", version.ref = "room" }
room-ktx = { module = "androidx.room:room-ktx", version.ref = "room" }
room-compiler = { module = "androidx.room:room-compiler", version.ref = "room" }
lifecycle-process = { module = "androidx.lifecycle:lifecycle-process", version.ref = "lifecycle" }
datastore-preferences = { module = "androidx.datastore:datastore-preferences", version.ref = "datastore" }

[plugins]
android-app = { id = "com.android.application", version.ref = "agp" }
android-lib = { id = "com.android.library", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
kotlin-kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kotlin" }
```

---

## 附录 B: SSE 封装参考骨架

```kotlin
// core-network/src/main/java/.../network/SseClient.kt
class SseClient @Inject constructor(
    private val client: OkHttpClient,
    private val json: Json,
) {
    fun chatStream(request: ChatRequest): Flow<ChatEvent> = callbackFlow {
        val factory = EventSources.createFactory(client)
        val req = Request.Builder()
            .url("$BASE_URL/chat")
            .post(json.encodeToString(request).toRequestBody(JSON_MEDIA))
            .build()

        val listener = object : EventSourceListener() {
            override fun onEvent(source: EventSource, id: String?, type: String?, data: String) {
                trySend(ChatEvent.fromRaw(type ?: return, data))
            }
            override fun onClosed(source: EventSource) { close() }
            override fun onFailure(source: EventSource, t: Throwable?, response: Response?) {
                close(t)
            }
        }

        val es = factory.newEventSource(req, listener)
        awaitClose { es.cancel() }
    }
}

sealed interface ChatEvent {
    data class Delta(val text: String) : ChatEvent
    data object BubbleEnd : ChatEvent
    data class ToolCall(val name: String, val args: String) : ChatEvent
    data class ToolResult(val name: String, val content: String) : ChatEvent
    data class Image(val id: String, val url: String) : ChatEvent
    data object Done : ChatEvent
    data class Error(val message: String) : ChatEvent

    companion object {
        fun fromRaw(type: String, data: String): ChatEvent = /* json parse */ TODO()
    }
}
```

**骨架不是实现**,接手 Claude 按它扩展,并为测试抽出接口。

---

## 文档版本

- v1.0 (2026-04-23): 初版,基于后端 Phase 2c(tag `v0.4-phase2c`)的真实契约编写。

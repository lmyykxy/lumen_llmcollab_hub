# 安卓前端大模型角色记忆

## 1. 前端职责

负责 Kotlin + Jetpack Compose 实现、路由、ViewModel、Room、DataStore、Coil、真机适配、Edge-to-edge。

---

## 2. 当前 MVP 屏幕

```text
Splash
Hero
Chat
Generated Image Chat
Moments
Settings
```

当前正式入口以 Chat 为中心，但要预留：

```text
Diary
DiaryDetail
Album
ImageDetail
Room
LifeFeed
```

---

## 3. 当前重要技术规则

```text
Splash 必须用户点击“轻轻推门”，不要自动跳转。
用户可见名用“陆小七”，不要主标题显示 Lumen。
React 视觉稿里的状态栏是假的，Compose 真机不能画假状态栏。
所有已接真后端的 Chat / Upload / Messages 不回退 mock。
预留功能可 mock，但不强制进入正式入口。
```

---

## 4. quote_ref 当前 PM 拍板

```text
POST /chat 保持 message 字段。
quote_ref P0 只支持 type=message。
sender_name snapshot：assistant=小七，user=你。
同 user_id 并发 /chat 后端返回 409 conversation_busy。
quote_ref 作为当前 user message prefix 注入模型。
本期不做 event_log / memory extractor。
```

---

## 5. 前端禁区

```text
不要新增 Login 作为正式 MVP
不要新增底部 Tab
不要新增好感度页
不要把 Hero 做成普通 landing page
不要把 Splash 改成倒计时
```

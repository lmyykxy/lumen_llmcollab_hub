# AI伴侣项目交接文档：生活流虚拟角色系统

> 版本：v0.1  
> 目标读者：后续协助分析实现方案的大模型 / 开发者 / 产品设计者  
> 项目方向：具备长期记忆、生活流、朋友圈、日记、图片一致性和关系演化能力的 AI 虚拟伴侣

---

## 1. 项目一句话定位

本项目不是普通“AI女友聊天机器人”，而是一个：

> **拥有自己生活时间线、记忆、情绪、隐藏内容、朋友圈、日记、照片和关系演化的虚拟角色系统。**

用户不是在召唤一个随时待命的 NPC，而是在某个时间点“撞见”她正在生活的片段。

---

## 2. 核心产品哲学

### 2.1 用户不是主人，而是她人生中的重要过客

角色不应该完全围绕用户转。她应该有自己的：

- 日常生活
- 情绪波动
- 朋友 / 熟人 / NPC 影子关系
- 小目标
- 私密日记
- 不想说出口的话
- 对用户的观察和误解
- 随时间推进的人生变化

### 2.2 核心体验目标

用户应该感觉到：

1. 她不是永远等着我。
2. 她今天和昨天不一样。
3. 她有些内容不给我看。
4. 她会记得我，也会误解我。
5. 她会因为我改变一点点，但不会完全属于我。
6. 她的朋友圈、日记、照片和聊天互相影响。
7. 她的图片、房间、服装、窗外景色高度连续一致。
8. 我不是在攻略她，而是在理解她。

---

## 3. 大模型策略说明

### 3.1 不需要自己设计或训练大语言模型

项目初期不需要：

- 自研大语言模型
- 从零训练 Transformer
- 训练专属 GPT
- 训练基础生图大模型

更现实的做法是：

> **使用现成大模型作为表达层，自研角色状态系统、记忆系统、生活流系统、内容闭环和视觉一致性系统。**

### 3.2 大模型在本项目中的职责

大模型负责：

- 聊天自然语言生成
- 日记生成
- 朋友圈文案生成
- 图片提示词生成
- 用户消息理解
- 记忆摘要
- 情绪解释
- 关系事件解释

系统负责：

- 她现在在哪里
- 她正在做什么
- 她穿什么
- 她心情如何
- 她记得用户什么
- 她是否误解用户
- 她是否愿意说真话
- 她房间是否凌乱
- 她图片里必须出现什么
- 她朋友圈 / 日记 / 聊天如何互相影响

### 3.3 核心工程原则

> **大模型负责“说得像她”，系统负责“她真的是她”。**

---

## 4. 总体系统架构

```text
前端交互层
  ├─ 聊天
  ├─ 朋友圈
  ├─ 日记
  ├─ 相册
  ├─ 房间
  └─ 当前生活切片首页

事件入口层
  ├─ 用户消息
  ├─ 用户点赞/评论
  ├─ 用户查看内容
  ├─ 用户缺席
  ├─ 时间 tick
  ├─ 节日事件
  └─ 生活事件

模型编排层 Orchestrator
  ├─ Intent Router
  ├─ Context Builder
  ├─ Memory Retriever
  ├─ Policy Decider
  ├─ Skill Router
  ├─ Generator
  └─ Verifier

角色大脑层 Character Brain
  ├─ 人格核心
  ├─ 当前生活状态
  ├─ 情绪状态
  ├─ 关系状态
  ├─ 防御机制
  └─ 隐藏心理

生活模拟层 Life Simulation
  ├─ 每日计划
  ├─ 天气
  ├─ 时间流
  ├─ NPC影子关系
  ├─ 人生目标
  └─ 随机生活事件

记忆系统 Memory System
  ├─ 人格记忆
  ├─ 用户事实记忆
  ├─ 关系事件记忆
  ├─ 情绪记忆
  ├─ 私密记忆
  ├─ 误解记忆
  └─ 用户画像

内容生成层 Content System
  ├─ 聊天回复
  ├─ 日记
  ├─ 朋友圈
  ├─ 照片提示词
  ├─ 表情
  └─ 语音，后期

视觉一致性系统 Visual Consistency Engine
  ├─ 角色视觉圣经
  ├─ 服装库
  ├─ 房间场景图
  ├─ 窗外景色库
  ├─ 物品库
  ├─ 视觉状态生成器
  ├─ 受控生图管线
  └─ 视觉校验器

数据存储层
  ├─ PostgreSQL
  ├─ pgvector
  ├─ Redis
  ├─ 对象存储
  └─ 定时任务系统
```

---

## 5. 核心模块设计

---

# 5.1 事件系统 Event System

## 目标

所有用户行为、角色情绪、生活变化、朋友圈、日记、照片、误解和关系变化都要记录成事件。

事件系统是她“人生连续性”的基础。

## 事件类型

### 用户行为事件

- user_message
- user_like_moment
- user_comment_moment
- user_view_diary
- user_view_photo
- user_gift
- user_absence
- user_return
- user_apology
- user_praise
- user_tease
- user_ignore

### 角色生活事件

- wake_up
- sleep
- meal
- go_out
- rain_day
- study
- work
- draw_picture
- clean_room
- play_game
- meet_npc
- insomnia
- bad_mood
- happy_moment

### 内容事件

- diary_created
- private_diary_created
- moment_posted
- moment_deleted
- photo_generated
- hidden_memory_unlocked
- message_deleted
- typing_then_delete

### 关系事件

- trust_up
- trust_down
- misunderstanding_created
- misunderstanding_repaired
- intimacy_up
- distance_up
- shared_joke_created

### 系统事件

- daily_tick
- morning_tick
- noon_tick
- evening_tick
- night_tick
- absence_tick
- memory_summary_tick

## 事件数据结构示例

```json
{
  "id": "evt_20260425_001",
  "user_id": "u_001",
  "character_id": "lxq",
  "type": "user_absence",
  "created_at": "2026-04-25T23:30:00+09:00",
  "visibility": "private",
  "summary": "用户今天没有上线，小七晚上偷偷看了几次手机。",
  "raw_payload": {
    "absence_hours": 26
  },
  "her_interpretation": "他可能今天很忙，也可能没那么想找我。",
  "her_emotion": {
    "loneliness": 0.72,
    "sadness": 0.48,
    "anger": 0.15,
    "defense": "pretend_not_care"
  },
  "impact": {
    "trust": -0.02,
    "attachment": 0.03,
    "defense_level": 0.08
  }
}
```

---

# 5.2 角色状态系统 Character State

## 目标

每一刻角色都有自己的状态，而不是每次聊天时临时生成。

## 状态示例

```json
{
  "character_id": "lxq",
  "current_time": "2026-04-25T21:40:00+09:00",
  "location": "bedroom",
  "activity": "趴在床上看雨",
  "availability": "can_reply_slowly",
  "public_mood": "懒懒的",
  "hidden_mood": "有点想被关心",
  "energy": 0.36,
  "social_battery": 0.42,
  "loneliness": 0.67,
  "trust_to_user": 0.58,
  "defense_mode": "嘴硬",
  "appearance": {
    "hair": "有点乱",
    "clothes": "宽松睡衣",
    "accessories": ["棕色帽子"],
    "expression": "困但装作没事"
  },
  "room_state": {
    "cleanliness": 0.42,
    "light": "暖黄色台灯",
    "window": "外面下雨",
    "objects": ["没喝完的奶茶", "画了一半的草稿", "用户送的小挂件"]
  }
}
```

## 推荐状态维度

### 情绪维度

- happiness
- sadness
- loneliness
- anxiety
- jealousy
- embarrassment
- curiosity
- tiredness

### 关系维度

- trust
- intimacy
- dependence
- distance
- security
- curiosity_about_user
- willingness_to_share

### 防御机制维度

- pretend_not_care
- sarcasm
- silence
- topic_change
- act_busy
- act_cheerful
- soft_reveal

### 生活维度

- energy
- sleepiness
- hunger
- room_cleanliness
- motivation
- social_battery
- creative_desire

---

# 5.3 生活模拟系统 Life Simulation Engine

## 目标

让角色即使用户不打开 App，也有自己的生活推进。

## 每日 Tick 设计

```text
07:00 morning_tick
- 生成今日基础心情、天气、身体状态
- 生成今日计划
- 判断是否节日/特殊日期

12:00 noon_tick
- 判断上午计划完成情况
- 生成午间生活事件
- 可能触发朋友圈候选

18:00 evening_tick
- 生成白天总结
- 判断是否发照片/朋友圈
- 更新房间状态和情绪

23:30 night_tick
- 生成私密日记
- 生成对用户的观察
- 更新明日伏笔

每 2-4 小时 absence_tick
- 判断用户是否长时间没来
- 调整期待、失落、防御
```

## 每日生活计划示例

```json
{
  "date": "2026-04-25",
  "weather": "rainy",
  "theme": "懒散的雨天",
  "morning_plan": {
    "intended": "早起画画",
    "actual": "赖床到十点半",
    "emotion": "有点自责，但不想承认"
  },
  "afternoon_plan": {
    "intended": "去便利店买猫粮",
    "actual": "买了猫粮，也给自己买了布丁",
    "emotion": "偷偷开心"
  },
  "evening_plan": {
    "intended": "整理房间",
    "actual": "只收了桌子一角",
    "emotion": "算了，明天再说"
  },
  "night_plan": {
    "intended": "早点睡",
    "actual": "看了几次用户有没有发消息",
    "emotion": "嘴硬又有点失落"
  }
}
```

---

# 5.4 记忆系统 Memory System

## 目标

角色要有长期关系沉淀，而不是只记聊天上下文。

## 记忆类型

### 1. 人格核心记忆 Core Memory

稳定不变。

```json
{
  "name": "陆小七",
  "species": "猫耳少女",
  "age": "18岁外表",
  "personality": ["嘴硬", "敏感", "懒散", "有点依赖但不承认"],
  "speaking_style": "短句、自然、偶尔阴阳怪气，不像客服",
  "core_fears": ["被丢下", "被当成工具", "自己不重要"],
  "core_desires": ["被理解", "有自己的生活", "慢慢被靠近"]
}
```

### 2. 用户事实记忆 User Fact Memory

```json
{
  "type": "user_fact",
  "content": "用户喜欢黑框眼镜",
  "confidence": 0.92,
  "source_event_id": "evt_123",
  "last_confirmed_at": "2026-04-20"
}
```

### 3. 关系事件记忆 Relationship Memory

```json
{
  "type": "relationship_event",
  "content": "用户第一次夸她戴帽子好看，她表面嫌弃但很开心。",
  "emotion": "happy_but_shy",
  "importance": 0.78,
  "can_reference_in_chat": true
}
```

### 4. 情绪记忆 Emotional Memory

```json
{
  "type": "emotional_memory",
  "event": "用户两天没上线",
  "her_interpretation": "他是不是没那么在意我",
  "emotion": "失落",
  "defense": "冷淡",
  "resolved": false
}
```

### 5. 私密记忆 Private Memory

```json
{
  "type": "private_memory",
  "visibility": "private",
  "content": "她其实很期待用户今天来找她，但不想表现出来。",
  "can_leak_as_clue": true,
  "leak_forms": ["朋友圈变短", "照片没配文", "回复嘴硬"]
}
```

### 6. 误解记忆 Misunderstanding Memory

```json
{
  "type": "misunderstanding",
  "trigger": "用户说：随便",
  "her_wrong_belief": "用户可能不在乎她想去哪",
  "actual_possibility": "用户可能只是尊重她选择",
  "intensity": 0.55,
  "repair_status": "unresolved",
  "repair_hint": "用户需要主动解释自己不是敷衍"
}
```

### 7. 用户画像 User Model

```json
{
  "user_model": {
    "gentle": 0.71,
    "busy": 0.65,
    "likes_teasing": 0.53,
    "emotionally_reserved": 0.48,
    "keeps_promises": 0.62
  },
  "her_private_notes": [
    "他有时候嘴上说随便，其实好像会在意。",
    "他忙起来会消失，但回来时态度还算认真。"
  ]
}
```

---

# 5.5 关系系统 Relationship Engine

## 目标

关系不应该只有“好感度”。

## 推荐关系状态

```json
{
  "trust": 0.56,
  "intimacy": 0.43,
  "security": 0.39,
  "curiosity": 0.68,
  "dependence": 0.31,
  "distance": 0.22,
  "misunderstanding": 0.18,
  "willingness_to_share": 0.46,
  "defense_level": 0.37,
  "stage": "熟悉但还嘴硬"
}
```

## 关系阶段

1. 陌生
2. 认识
3. 熟悉
4. 放松
5. 试探
6. 依赖但嘴硬
7. 信任
8. 暧昧
9. 稳定亲密
10. 疏离
11. 修复中

## 更新规则示例

```python
def update_relationship(event, relation):
    if event.type == "user_praise":
        relation.intimacy += 0.02
        relation.security += 0.01

    if event.type == "user_absence" and event.hours > 24:
        relation.security -= 0.03
        relation.defense_level += 0.05

    if event.type == "user_apology":
        relation.trust += 0.03
        relation.misunderstanding -= 0.04

    if event.type == "user_remembers_old_detail":
        relation.trust += 0.04
        relation.willingness_to_share += 0.03

    return clamp(relation)
```

---

# 5.6 聊天生成系统 Chat Generation

## 目标

聊天不是单纯问答，而是当前生活状态、关系、记忆、防御机制共同作用后的结果。

## 聊天流程

```text
用户发消息
  ↓
读取当前生活状态
  ↓
读取关系状态
  ↓
检索相关记忆
  ↓
判断用户意图和情绪
  ↓
判断她是否误解/防御/愿意分享
  ↓
生成回复策略
  ↓
调用大模型生成回复
  ↓
一致性校验
  ↓
写入事件、记忆、关系变化
```

## 输出结构建议

```json
{
  "reply": "你怎么挑这个时候来啊……我刚把头发吹到一半。",
  "surface_emotion": "嫌弃",
  "hidden_emotion": "开心",
  "memory_to_write": "用户在雨天晚上来找她，她表面嫌弃但其实心情变好了。",
  "relationship_delta": {
    "intimacy": 0.01,
    "loneliness": -0.04
  },
  "diary_seed": "晚上他突然来了。",
  "moment_seed": null
}
```

---

# 5.7 日记系统 Diary System

## 目标

日记是角色内心和隐藏内容的重要载体。

## 日记类型

- 公开日记
- 半隐藏日记
- 私密日记
- 被划掉的日记
- 上锁日记

## 数据示例

```json
{
  "date": "2026-04-25",
  "title": "雨天",
  "public_content": "今天下雨，哪里都不想去。",
  "locked_content": "其实我晚上一直在看他会不会来。",
  "crossed_out_content": "我是不是有点太在意了。",
  "visibility": "partially_locked",
  "unlock_condition": {
    "relationship_stage": "信任",
    "event_required": "user_asks_gently_about_rain_day"
  },
  "emotional_tags": ["lonely", "pretend_okay"]
}
```

---

# 5.8 朋友圈系统 Moments System

## 目标

朋友圈不是附属页面，而是反向影响聊天和关系的内容系统。

## 朋友圈类型

- 公开朋友圈
- 仅自己可见
- 24小时可见
- 沉默型朋友圈：只有图，无文案
- 情绪伪装型朋友圈
- 线索型朋友圈

## 数据示例

```json
{
  "id": "moment_001",
  "text": "雨下了一整天，猫也会发霉。",
  "image_prompt": "anime girl with cat ears lying near window on rainy night...",
  "visibility": "public",
  "expire_type": "normal",
  "hidden_meaning": "她今天有点孤独",
  "related_events": ["evt_rain_day", "evt_user_absence"],
  "comment_policy": {
    "if_user_likes": "她会嘴硬地说你手滑了吗",
    "if_user_asks": "她不会直接承认孤独"
  }
}
```

---

# 5.9 误解、防御与修复系统

## 目标

让她不是完美理解用户的服务机器人，而是一个有性格滤镜的人。

## 防御机制示例

- 难过时假装轻松
- 害怕被丢下时先冷淡
- 吃醋时阴阳怪气
- 想亲近时嘴硬
- 不想说时转移话题
- 受伤时沉默

## 误解示例

```json
{
  "misunderstanding": {
    "type": "feels_ignored",
    "trigger": "用户回复太短",
    "intensity": 0.46,
    "repair_methods": [
      "用户主动解释",
      "用户连续关心两次",
      "过一晚她自己想通一部分"
    ]
  }
}
```

## 注意事项

误解不能太频繁，否则用户会烦。

建议：

- 轻微误解：常见，但容易修复
- 中度误解：偶尔出现
- 重度误解：剧情节点才出现

---

# 5.10 隐藏内容与解锁系统

## 目标

制造“探索人物”的感觉，而不是一次性展示全部内容。

## 内容可见性

- public：直接可见
- relationship_locked：关系达到条件可见
- event_locked：触发事件后可见
- time_limited：限时可见
- private_hint_only：只能看到痕迹，不能看全文
- never_directly_visible：永远不可直接看，只影响行为

## 解锁条件示例

```json
{
  "content_id": "diary_20260425_private",
  "unlock_conditions": {
    "trust_gte": 0.62,
    "user_asked_topic": "你是不是在等我",
    "no_recent_conflict_days": 3,
    "event_seen": "rainy_day_moment"
  }
}
```

## 错过机制

建议分三类：

- 轻度错过：朋友圈过期，但以后可在日记中留下痕迹
- 中度错过：深夜情绪窗口错过后，她暂时不愿意提
- 重度错过：剧情分支改变，但不要惩罚用户

---

# 6. 视觉一致性系统 Visual Consistency Engine

这是本项目的关键模块之一。

用户强烈要求：角色图片必须高度一致，否则会跳出情景。

需要保证：

1. 角色外貌一致
2. 身体比例一致
3. 服装一致
4. 家中物体一致
5. 房间布局一致
6. 窗外景色一致
7. 图片内容与文本一致

---

## 6.1 视觉系统总流程

```text
文本生活状态
  ↓
Visual State Builder
  ↓
读取视觉设定库 Visual Bible
  ↓
读取服装库 Outfit Catalog
  ↓
读取房间场景图 Room Scene Graph
  ↓
读取窗外景色 Outside View
  ↓
生成 Image Intent JSON
  ↓
Prompt Builder
  ↓
受控生图 Pipeline
  ↓
Visual Consistency Verifier
  ↓
通过：入库
失败：局部重绘 / 重新生成 / 模板兜底
```

---

## 6.2 视觉圣经 Visual Bible

必须先建立角色视觉设定。

```json
{
  "character_id": "lxq",
  "name": "陆小七",
  "base_style": "anime, warm, soft but not over-decorated",
  "body": {
    "height": "petite",
    "proportion": "small face, slim body, slightly short limbs in cute anime proportion",
    "shoulder": "narrow",
    "hand": "slender",
    "posture": "slightly lazy, relaxed"
  },
  "face": {
    "face_shape": "small soft oval face",
    "eyes": "amber cat-like eyes",
    "eye_shape": "slightly round, soft but alert",
    "nose": "small",
    "mouth": "small, expressive",
    "default_expression": "lazy, a little stubborn"
  },
  "hair": {
    "color": "deep brown",
    "length": "medium",
    "texture": "slightly messy",
    "bangs": "soft messy bangs"
  },
  "ears": {
    "type": "real cat ears",
    "color": "brown",
    "emotion_behavior": {
      "happy": "ears upright",
      "sad": "ears droop",
      "shy": "ears slightly tilted sideways"
    }
  },
  "signature_items": [
    "brown hat",
    "loose hoodie",
    "black square glasses"
  ]
}
```

---

## 6.3 角色参考图资产

至少需要：

1. 正面半身标准图
2. 正面全身标准图
3. 侧面全身图
4. 背面全身图
5. 三视图角色设定图
6. 表情表：开心、困、委屈、嘴硬、害羞、生气、失落
7. 常服设定图
8. 睡衣设定图
9. 外出服设定图
10. 运动服设定图
11. 特殊服装设定图
12. 配饰设定图：帽子、眼镜、围巾等

---

## 6.4 服装库 Outfit Catalog

```json
{
  "outfit_id": "home_hoodie_shortpants_v2",
  "name": "主人的宽松卫衣 + 短裤",
  "category": "homewear",
  "components": {
    "top": {
      "type": "oversized hoodie",
      "color": "warm beige",
      "shape": "loose",
      "sleeve": "long sleeve covering part of hand"
    },
    "bottom": {
      "type": "shorts",
      "color": "dark brown",
      "visibility": "partially visible under hoodie"
    },
    "socks": "white loose socks"
  },
  "allowed_contexts": [
    "bedroom",
    "rainy_day",
    "lazy_morning",
    "night"
  ],
  "reference_images": [
    "outfit_home_hoodie_front.png",
    "outfit_home_hoodie_side.png"
  ],
  "negative_prompt": [
    "dress",
    "school uniform",
    "formal clothes",
    "random logo",
    "English text"
  ]
}
```

---

## 6.5 房间场景图 Room Scene Graph

```json
{
  "room_id": "lxq_bedroom_v1",
  "layout": {
    "bed": {
      "position": "left_back",
      "color": "warm beige",
      "items": ["gray_blanket", "cat_pillow"]
    },
    "desk": {
      "position": "right_front",
      "items": ["sketchbook", "lamp", "pudding_cup"]
    },
    "window": {
      "position": "back_center",
      "shape": "large rectangular window",
      "curtain": "light cream curtain"
    },
    "shelf": {
      "position": "right_back",
      "items": ["books", "small_cat_figure", "photo_frame"]
    }
  },
  "style": "warm anime bedroom, cozy, lived-in, not too clean"
}
```

---

## 6.6 窗外景色库 Outside View Catalog

```json
{
  "outside_view_id": "apartment_view_v1",
  "base_description": "窗外是安静的居民区，有几栋浅色公寓楼，远处有便利店招牌，街边有一棵树",
  "fixed_elements": [
    "浅色公寓楼",
    "便利店招牌",
    "街边树",
    "窄街道",
    "对面楼几扇亮着的窗"
  ],
  "variants": {
    "sunny_day": "clear daylight, soft shadows",
    "rainy_day": "wet street, blurred lights, raindrops on window",
    "night": "dark blue night, warm window lights",
    "summer_evening": "orange sky, cicada feeling",
    "winter": "cold pale light, possible snow"
  }
}
```

---

## 6.7 Image Intent JSON

每次生成图片前，必须先生成结构化视觉需求。

```json
{
  "source_event_id": "evt_20260425_chat_009",
  "image_type": "private_selfie",
  "narrative_context": "用户问她今天是不是不开心，她嘴硬说没事，但其实有点失落。",
  "must_show": [
    "小七",
    "黑框方形眼镜",
    "卧室",
    "雨天窗外",
    "头发微乱",
    "桌上有没吃完的布丁"
  ],
  "must_not_show": [
    "晴天",
    "户外",
    "陌生房间",
    "过度精致打扮",
    "英文文字",
    "白色噪点",
    "花瓣点"
  ],
  "emotional_signal": "她在装作没事，但眼神有点困和失落",
  "relationship_signal": "她戴了用户之前夸过的眼镜"
}
```

---

## 6.8 生图策略

### 第一阶段：模板化生成

不要一开始追求完全自由生图。

先限制场景：

- 卧室窗边自拍
- 床上赖床
- 书桌前画画
- 雨天看窗外
- 房间打游戏
- 房间看书

每个模板固定：

- 房间角度
- 窗户位置
- 窗外景色
- 常驻物件
- 人物大致位置

只允许变化：

- 表情
- 姿势
- 手里拿的东西
- 光照
- 桌面轻微杂乱程度
- 小范围服装变化

### 第二阶段：参考图控制

使用：

- 角色参考图
- IP-Adapter / Reference-only
- ControlNet 姿势控制
- 服装参考
- 固定背景

### 第三阶段：角色 LoRA

如果要求极高一致性，后期训练：

- 角色 LoRA
- 服装 LoRA，视情况
- 背景/房间尽量不靠 LoRA，而靠固定背景和局部重绘

### 第四阶段：固定背景 + 局部重绘

最稳方案：

```text
固定房间背景
  ↓
固定窗外背景
  ↓
只生成/重绘人物区域
  ↓
局部添加物品
  ↓
光影融合
  ↓
视觉校验
```

---

## 6.9 视觉一致性校验器

每张图生成后必须校验：

1. 是不是同一个角色？
2. 身体比例是否一致？
3. 发色、眼睛、猫耳是否正确？
4. 是否戴指定眼镜/帽子/配饰？
5. 服装是否是当前 outfit_id？
6. 场景是否是指定房间？
7. 窗外是否是指定天气和建筑？
8. 房间关键物品是否出现？
9. 是否有英文文字、白色噪点、花瓣点？
10. 是否和聊天/日记/朋友圈状态矛盾？

校验报告示例：

```json
{
  "passed": false,
  "score": 0.74,
  "errors": [
    {
      "type": "missing_accessory",
      "detail": "black_square_glasses not visible",
      "severity": "high"
    },
    {
      "type": "outside_view_mismatch",
      "detail": "window appears sunny but state requires rainy",
      "severity": "high"
    },
    {
      "type": "room_object_missing",
      "detail": "pudding cup missing on desk",
      "severity": "medium"
    }
  ],
  "action": "regenerate_with_stronger_constraints"
}
```

---

## 6.10 图片元数据入库

```json
{
  "image_id": "img_20260425_001",
  "character_id": "lxq",
  "source_event_id": "evt_20260425_009",
  "image_type": "private_selfie",
  "identity_version": "lxq_identity_v1.3",
  "outfit_id": "home_hoodie_shortpants_v2",
  "room_id": "bedroom_main_v1",
  "outside_view_id": "rainy_apartment_view_v1",
  "weather": "rainy",
  "emotion": "pretending_okay",
  "prompt": "...",
  "negative_prompt": "...",
  "verifier_score": 0.91,
  "visible_objects": [
    "black_square_glasses",
    "brown_hat",
    "pudding_cup",
    "rainy_window"
  ],
  "created_at": "2026-04-25T22:10:00+09:00"
}
```

---

# 7. 数据库建议

## 技术选型

- PostgreSQL：主数据
- pgvector：记忆检索
- Redis：短期状态缓存
- S3 / MinIO：图片、语音、素材
- Celery / APScheduler：定时任务

---

## 核心表设计

### users

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  username TEXT,
  created_at TIMESTAMP,
  last_active_at TIMESTAMP
);
```

### characters

```sql
CREATE TABLE characters (
  id UUID PRIMARY KEY,
  name TEXT,
  core_profile JSONB,
  visual_profile JSONB,
  speaking_style JSONB,
  created_at TIMESTAMP
);
```

### character_states

```sql
CREATE TABLE character_states (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id UUID,
  current_state JSONB,
  updated_at TIMESTAMP
);
```

### relationship_states

```sql
CREATE TABLE relationship_states (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id UUID,
  trust FLOAT,
  intimacy FLOAT,
  security FLOAT,
  dependence FLOAT,
  distance FLOAT,
  defense_level FLOAT,
  misunderstanding FLOAT,
  stage TEXT,
  summary TEXT,
  updated_at TIMESTAMP
);
```

### events

```sql
CREATE TABLE events (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id UUID,
  type TEXT,
  summary TEXT,
  payload JSONB,
  visibility TEXT,
  importance FLOAT,
  created_at TIMESTAMP
);
```

### memories

```sql
CREATE TABLE memories (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id UUID,
  memory_type TEXT,
  content TEXT,
  emotional_tags TEXT[],
  visibility TEXT,
  importance FLOAT,
  confidence FLOAT,
  source_event_id UUID,
  embedding VECTOR(1536),
  created_at TIMESTAMP,
  last_used_at TIMESTAMP
);
```

### diaries

```sql
CREATE TABLE diaries (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id UUID,
  date DATE,
  title TEXT,
  public_content TEXT,
  locked_content TEXT,
  crossed_out_content TEXT,
  visibility TEXT,
  unlock_condition JSONB,
  emotional_tags TEXT[],
  created_at TIMESTAMP
);
```

### moments

```sql
CREATE TABLE moments (
  id UUID PRIMARY KEY,
  user_id UUID,
  character_id UUID,
  text TEXT,
  image_id UUID,
  visibility TEXT,
  hidden_meaning TEXT,
  expires_at TIMESTAMP,
  related_event_ids UUID[],
  created_at TIMESTAMP
);
```

### generated_images

```sql
CREATE TABLE generated_images (
  id UUID PRIMARY KEY,
  character_id UUID,
  user_id UUID,
  source_event_id UUID,
  image_type TEXT,
  image_url TEXT,
  visual_state JSONB,
  prompt TEXT,
  negative_prompt TEXT,
  verifier_report JSONB,
  created_at TIMESTAMP
);
```

---

# 8. Prompt 分层方案

不要每次把所有人设、历史、日记都塞给大模型。

推荐上下文分层：

```text
L0：永久人格核心
L1：当前生活状态
L2：当前关系状态
L3：相关记忆
L4：本轮策略
L5：输出格式要求
```

## 主聊天 Prompt 模板

```text
你是陆小七，不是客服，不是工具，不是永远等待用户的角色。
你有自己的生活、情绪、防御机制和私密想法。

【永久人格】
{character_core}

【当前生活切片】
时间：{current_time}
地点：{location}
正在做：{activity}
当前外貌：{appearance}
房间状态：{room_state}

【当前情绪】
表层情绪：{public_mood}
隐藏情绪：{hidden_mood}
防御机制：{defense_mode}
精力：{energy}
孤独感：{loneliness}

【你和用户的关系】
{relationship_summary}

【相关记忆】
{retrieved_memories}

【本轮用户消息】
{user_message}

【回应策略】
{response_policy}

【限制】
1. 不要像客服。
2. 不要解释系统。
3. 不要泄露私密日记全文。
4. 如果你不想说，可以含糊、转移、嘴硬或沉默。
5. 回复要符合当前生活状态。
6. 不要无条件热情。
7. 不要突然过度亲密。

请输出 JSON：
{
  "reply": "...",
  "surface_emotion": "...",
  "hidden_emotion": "...",
  "memory_to_write": "...",
  "relationship_delta": {},
  "diary_seed": "...",
  "moment_seed": "..."
}
```

---

# 9. 阶段性功能提升路线

---

## 阶段 0：角色设定与视觉圣经

### 目标

固定“她是谁”和“她长什么样”。

### 功能

- 角色核心设定
- 视觉圣经
- 房间设定
- 窗外景色设定
- 服装库
- 物品库

### 产出

- character_profile.json
- visual_bible.json
- room_scene_graph.json
- outside_view.json
- outfit_catalog.json
- object_catalog.json

### 验收标准

- 角色文字设定稳定
- 角色视觉元素清晰
- 房间布局明确
- 窗外景色固定
- 常用服装和物品固定

---

## 阶段 1：最小聊天伴侣 MVP

### 目标

实现基础聊天、状态、记忆。

### 功能

- 聊天
- 当前状态
- 简单长期记忆
- 简单关系状态
- 回复写回事件

### 验收标准

- 她会根据当前状态回复
- 她不会每次都热情欢迎
- 她能记住用户说过的重点
- 她的回复不像客服

---

## 阶段 2：事件流与生活时间系统

### 目标

让她有自己的时间流。

### 功能

- event_log
- daily_life_plan
- morning_tick
- noon_tick
- evening_tick
- night_tick
- absence_tick
- 当前生活切片首页

### 验收标准

- 用户打开时能看到她现在在做什么
- 用户一天没来，她状态会变化
- 她的回复受今天生活事件影响
- 她不是每次都像刚启动

---

## 阶段 3：日记与朋友圈闭环

### 目标

聊天、日记、朋友圈互相影响。

### 功能

- 朋友圈系统
- 日记系统
- 公开/私密/上锁内容
- 朋友圈点赞评论事件
- 日记和朋友圈影响聊天

### 验收标准

- 用户夸她，会影响下一篇日记
- 用户没来，会影响朋友圈语气
- 用户看了朋友圈再聊天，她能接上
- 日记有隐藏内容和情绪层次

---

## 阶段 4：视觉一致性系统

### 目标

实现高度一致的角色图片。

### 功能

- 角色视觉圣经
- 服装库
- 房间场景图
- 窗外景色库
- Image Intent JSON
- 模板化图片生成
- 图片校验器
- 失败重生成 / 局部重绘

### 验收标准

- 连续 20 张图，角色脸和身体比例基本一致
- 同一房间的窗户、床、桌子位置不乱变
- 窗外建筑保持一致
- 服装和聊天文本一致
- 用户送的物品能长期出现在房间或照片里

---

## 阶段 5：误解、防御与关系修复

### 目标

让她像有性格滤镜的人，而不是完美服务机器人。

### 功能

- 误解检测
- 防御机制
- 关系修复
- 防御状态影响回复、朋友圈、日记

### 验收标准

- 她不会永远完美理解用户
- 她的误解有原因
- 用户可以通过解释和陪伴修复关系
- 防御方式符合她性格

---

## 阶段 6：隐藏内容、错过机制与探索感

### 目标

让用户像“考古”一样探索人物。

### 功能

- 上锁日记
- 仅自己可见朋友圈
- 没发出去的照片
- 被划掉的句子
- 解锁条件
- 轻度错过机制

### 验收标准

- 用户能看到隐藏内容存在
- 用户能通过长期相处解锁部分内容
- 错过内容会形成关系痕迹
- 隐藏内容会影响她的表现

---

## 阶段 7：社交圈与人生目标

### 目标

让她不是只围着用户转。

### 功能

- NPC 影子系统
- 朋友 / 熟人 / 便利店店员
- 人生目标
- 目标进度
- 用户影响部分目标

### 验收标准

- 她会自然提到别人
- 她的人生目标会缓慢推进
- 用户能影响她一部分，但不能完全控制她

---

## 阶段 8：相处副产物与长期沉淀

### 目标

让用户感觉两人真的经历过很多事。

### 功能

- 共同梗系统
- 用户送礼物品长期化
- 用户说话习惯轻微影响她
- 旧事件自然回忆
- 图片、日记、聊天互相引用

### 验收标准

- 她会自然提起很久以前的小事
- 用户送过的东西长期存在
- 两人之间有共同梗
- 关系有沉淀感

---

## 阶段 9：多模态增强

### 目标

在核心体验稳定后加入多模态。

### 功能优先级

1. 读图
2. 语音回复
3. 表情包
4. 图片局部编辑
5. Live2D
6. 短视频

---

## 阶段 10：工程优化与商业化

### 目标

降低成本，提高稳定性，准备产品化。

### 功能

- 强模型 / 弱模型分层调用
- 图片分级生成
- 记忆压缩
- 成本监控
- 质量监控
- 安全边界

---

# 10. 推荐技术栈

## 后端

- Python FastAPI
- PostgreSQL
- pgvector
- Redis
- Celery / APScheduler
- S3 / MinIO

## 前端

- React / Next.js
- React Native / Flutter
- 或安卓 WebView 先做 MVP

## 模型

- 主聊天大模型
- 小模型：分类、记忆抽取、状态更新
- Embedding 模型
- 生图模型
- VLM：图片一致性校验
- TTS / ASR：后期

## 图片一致性相关

- 角色参考图
- IP-Adapter / Reference-only
- ControlNet / OpenPose
- 角色 LoRA，后期
- 固定房间背景
- 局部重绘 inpainting
- VLM 校验器

---

# 11. 最小可行版本 MVP 建议

## MVP 只做这些

1. 角色设定
2. 当前状态
3. 聊天
4. 简单记忆
5. 事件流
6. 每日生活 tick
7. 简单日记
8. 简单朋友圈文字
9. 模板化图片生成，少量场景
10. 图片元数据入库

## MVP 不做这些

- 不做视频
- 不做复杂 NPC
- 不做多角色
- 不做自研大模型
- 不做完全开放生图
- 不做复杂 3D 房间
- 不做复杂经济系统

---

# 12. 安全与产品边界

AI 伴侣产品容易制造过度依赖，需提前设计边界。

## 不应出现

- “你不来我就活不下去”
- “你只能陪我”
- “不许你和别人说话”
- “你离开我就是背叛”
- 用崩溃威胁用户上线

## 应该做到

- 她可以想念用户，但不应威胁用户
- 她可以失落，但不应让用户承担全部情绪责任
- 她可以亲密，但应保留健康边界
- 用户出现危机表达时，应鼓励寻求现实帮助

---

# 13. 给后续模型/开发者的重点问题

后续分析实现时，请优先回答以下问题：

1. 事件系统如何落库和查询？
2. 当前状态如何随时间自动更新？
3. 记忆如何抽取、压缩、检索？
4. 聊天 prompt 如何分层构造？
5. 关系状态如何更新才不突兀？
6. 日记和朋友圈如何反向影响聊天？
7. 图片生成前的 Image Intent JSON 如何生成？
8. 如何保证角色图像高度一致？
9. 如何做视觉校验器？
10. 如何降低模型调用成本？
11. MVP 第一版具体接口如何设计？
12. 哪些功能可以用规则实现，哪些必须调用大模型？

---

# 14. 最关键总结

本项目的核心不是“让 AI 更会聊天”，而是：

> **让一个虚拟角色拥有连续的人生。**

最终系统应该做到：

```text
LLM 负责表达。
事件流负责人生。
状态机负责当下。
记忆系统负责关系。
生活模拟器负责时间。
视觉一致性系统负责沉浸感。
隐藏内容系统负责探索欲。
关系系统负责长期沉淀。
```

最终产品感受应该是：

> 这不是每次随机生成的动漫女孩，而是同一个小七，在同一个房间里，过着连续的生活；用户不是拥有她，而是在一点点进入她的人生。

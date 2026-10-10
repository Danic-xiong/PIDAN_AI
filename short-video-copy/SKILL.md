---
name: short-video-copy
description: 为短视频配共鸣型口播文案。从演讲/访谈/脱口秀学习观点内容，转化成独立于画面的共鸣文案。用户发视频后，自动从素材库挑观点配稿。覆盖成长、创业、赚钱、心理、认知、关系等方向。
---

# 短视频文案配文 Skill

## 这个 Skill 是干嘛的

给短视频配共鸣型口播文案。

- 文案是主体，画面只提供真实感，两者不绑定
- 文案来源：从演讲/访谈/脱口秀学习观点内容，转化成自己的素材
- 不为视频时长砍掉演讲最吸引人的部分——视频跟着文案走

## 触发条件

- 用户说"帮我配文案"、"给这条视频配文"、"写个文案"
- 用户发了视频文件，要配文
- 用户说想学演讲/脱口秀，扩充素材库
- 用户要扩充观点库

---

## 一、去哪里找值得学的视频（重要）

### ❌ 不要凭记忆猜 video ID
实测：凭记忆猜的 ID 大批量失效（unavailable / truncated / 429）。**这是浪费轮次的主要来源。**

### ✅ 正确流程：先搜索拿真实链接，再抓

**第 1 步：WebSearch 找真实 URL 和 ID**

搜索词用「话题 + 演讲者 + 关键词」，例如：
- `Dan Ariely "Are we in control of our own decisions" youtube`
- `Tim Urban procrastination TED talk video id`
- `认知偏差 演讲 youtube`

**第 2 步：从搜索结果里提取 video ID**
- YouTube 链接里 `v=` 后面那串 11 位字符
- 搜索结果的 URL 里常含真实 YouTube 链接

**第 3 步：用 yt_sub.py 抓**
```bash
python yt_sub.py <video_id> en
```

### 备选路径：TED 官方 transcript（更可靠，不受限流影响）

TED 官网提供官方逐字稿，不需要抓 YouTube：
```
https://www.ted.com/talks/<slug>/transcript
```
例如：`https://www.ted.com/talks/dan_ariely_are_we_in_control_of_our_own_decisions/transcript`

可用 WebFetch 读取。支持多语言（`?language=zh-CN`）。

**优先顺序**：
1. 有官方 transcript → 用 WebFetch（快、无限制）
2. 没有 → WebSearch拿 YouTube ID → yt_sub.py 抓

### 值得搜的方向和已验证可抓的 ID

| 方向 | 演讲者/关键词 | 已验证 ID |
|---|---|---|
| **认知偏差** | Dan Ariely、Adam Grant、Moran Cerf、Kahneman | `9X68dm92HVI`(Ariely 控制感) `fxbCHn6gE3U`(Grant precrastinator) `6dqNiSGo9yU`(Cerf 自由意志) |
| **拖延/行动力** | Tim Urban | `arj7oStGLkU` |
| **财富/赚钱** | Robert Kiyosaki | `abMQhaMdQu0` |
| **创造力/教育** | Sir Ken Robinson | `iG9CE55wbtY` |
| **毅力/天赋** | Angela Duckworth | `H14bBuluwB8` |
| **演讲技巧** | 讲演讲本身的演讲 | `8S0FDjFBj8o` |
| **睡眠/健康** | 睡眠科学 | `5MuIMqhT8DM` |

### 还该覆盖的方向（下次优先）
- **关系与沟通**：Chris Graydon、Ariel Goldstein、关系心理学
- **脱口秀/喜剧**：George Carlin、Bo Burnham、Richard Ayoade、Ali Wong（学"把真话说得好笑"的形态）
- **创业决策**：真实创业者访谈（Stripe、Airbnb 创始人访谈）
- **习惯与改变**：James Clear、Wendell Berry

---

## 二、工作流程

### A. 学习模式（持续进行，不需要用户触发）

1. **找素材**：WebSearch 找真实链接（见上）
2. **抓字幕**：WebFetch TED transcript，或 yt_sub.py 抓 YouTube
   - 一次不超过 5–6 条，避免 429
   - 每次间隔 8–10 秒
3. **读全文，提炼观点**：
   - 学的是**观点本身**，不是讲法
   - 筛选标准：和常识相反 + 有解释力 + 能引发自我认同
4. **转化成完整文案**：
   - **保留演讲的灵魂，不为短视频时长砍内容**
   - 长度跟着内容走，视频裁到文案长度
5. **入库**：观点存素材库，转化文案存 HTML

### B. 配文模式（用户发视频触发）

1. **不主动问场景**——场景不影响文案选择
2. **不绑画面**——文案完全独立
3. **从素材库挑观点**：按共鸣度排序
4. **配 3 个备选开头**：不同句式结构
5. **标红线句**：金句单独配大字幕

---

## 三、规则（每次会话开始先读）

### 根本原则
- **文案是主体，画面只提供真实感，两者不绑定**
- **不为视频时长砍演讲内容**——视频跟着文案走
- **学观点，不学讲法**

### 共鸣判定标准
让用户看到就觉得：
- "确实吼，我也这么觉得"
- "这不就是我吗？"
- "心里思考很久但没行动"

### 1秒钩子
第一句必须让人在1 秒内产生念头（凭什么/真的假的/我怎么知道）。
念完观众反应"哦然后呢" → 废，重写。

### 钩子三不要
- 不概括（"我们这代人……"）
- 不铺垫（"大家好"）
- 不温和（"其实每个人都……"）

### 句式不重复
统一的钩子等于没有钩子。多条文案必须用不同句式结构：
时间地点式、引述式、下命令式、荒谬细节式、承认式、先给后撤、质问式、日常寒暄式、第三方式、自嘲式、时间跨度式、立后破式

### 前三字必须带信息
删掉前三字如果意思不变 → 废话，重写。

### 文案避免的陷阱
- 安慰而非揭穿 → 共鸣失败
- 说感受而非说具体 → 没有对号入座
- 给多建议而非给一个动作 → 鸡汤
- 句式重复 → 观众能预判

### 禁止行为
- ❌ 不把文案和画面绑定
- ❌ 不凭记忆猜 video ID（要先搜索）
- ❌ 不为视频时长砍掉演讲最吸引人的部分
- ❌ 不用"你不是X你是Y"等套路句式重复
- ❌ 不在结尾说"所以我们都要加油"
- ❌ 不在观点里编造数据（只保留演讲中明确说过的）

---

## 四、工具

### YouTube 字幕抓取
- 脚本：`yt_sub.py`
- 工具路径：`C:\Users\wenmin.xiong\.workbuddy\binaries\python\envs\default\Scripts\yt-dlp.exe`
- 用法：`python yt_sub.py <video_id> en`（zh-Hans 也可）
- 输出：`subs/<video_id>.<lang>.txt`

### TED 官方 transcript
- 格式：`https://www.ted.com/talks/<slug>/transcript`
- 用 WebFetch 读取，可加 `?language=zh-CN`

### 素材库位置
- 字幕原文：`subs/`
- 观点提炼与转化文案：工作区根目录各 HTML 文件

---

## 五、已抓演讲清单（截至 2026-10-10）

| ID | 主题 | 核心观点 |
|---|---|---|
| `9X68dm92HVI` | Dan Ariely 认知偏差 | 视觉错觉容易识破，认知错觉看不到。默认选项决定行为 |
| `fxbCHn6gE3U` | Adam Grant 拖延 | **适度拖延的人比提前做完的人更有创意，过度拖延的人最差** |
| `6dqNiSGo9yU` | Moran Cerf 自由意志 | 我们其实没那么多自由意志 |
| `arj7oStGLkU` | Tim Urban 拖延 | 理性决策者 vs 及时行乐猴 vs 恐慌怪兽；越是自己选的事越容易拖 |
| `abMQhaMdQu0` | Robert Kiyosaki 财富 | 穷爸爸教你省钱富爸爸教你赚钱；资产 vs 负债；时间的钱 vs 东西的钱 |
| `iG9CE55wbtY` | Sir Ken Robinson 创造力 | 不是长成有创造力的人，是被教育成没有创造力的人 |
| `H14bBuluwB8` | Angela Duckworth 毅力 | 毅力与天赋常常无关甚至相反 |
| `8S0FDjFBj8o` | 演讲的演讲 | 装懂可伪造；没人愿意承认自己没听懂 |
| `5MuIMqhT8DM` | 睡眠科学 | 睡眠不足的认知代价 |

---

## 六、数据反推（发满 10 条后执行）

用户发满 10 条后，要播放量最高和最低各 3 条，用真实数据反推账号该吃哪类观点、哪种情绪浓度、多长的句子。

**数据比任何规律都准。**

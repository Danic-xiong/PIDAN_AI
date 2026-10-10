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

### 三条已验证路径（按稳定性排序）

| 路径 | 脚本 | 稳定性 | 说明 |
|---|---|---|---|
| **① TED 官方 transcript** | `ted_transcript.py` | ★★★★★ | 官方逐字稿，无字幕轨问题，无限流，含搜索 |
| **② YouTube + youtube-transcript-api** | `yt_search.py` | ★★★★☆ | 实测 4/4 成功，比 yt-dlp 稳 |
| **③ YouTube + yt-dlp** | `yt_sub.py` / `yt_search.py`降级 | ★★☆☆☆ | 容易触发 HTTP 429，仅作降级备用 |

---

### 路径①：TED 官方逐字稿（首选）

```bash
# 搜索 TED 讲座拿 slug
python ted_transcript.py --list "procrastination" 5

# 抓全文（自动存 subs/ted_<slug>.en.txt）
python ted_transcript.py sir_ken_robinson_do_schools_kill_creativity
```

- 实现原理：transcript 页面的 `__NEXT_DATA__` 里内嵌完整逐字稿
  （`props.pageProps.transcriptData.translation.paragraphs[].cues[].text`）
- 实测：Sir Ken Robinson 演讲拿到 **105 段 / 17678 字**完整稿
- **不要加 `?language=` 参数**——加了会破坏 `__NEXT_DATA__` 结构导致解析失败
- UA 必须用简短的 `Mozilla/5.0 (Windows NT 10.0; Win64; x64)`，带完整 Chrome UA 会被 403 `Forbidden-UA`
- TED-Ed 动画短片也在 TED 上，有逐字稿但较短（几千字）

---

### 路径②：YouTube 搜索 + transcript-api（主力 YouTube 路径）

```bash
# 只搜索候选
python yt_search.py "dan ariely decision"

# 搜索并抓取前 N 条字幕
python yt_search.py --grab "tim urban procrastination" --n 3
```

- 已内置 `search_youtube()`：解析 YouTube 搜索页 HTML 提取 videoId + 标题，**不需要 API key**
- 抓取走 `youtube-transcript-api`（已装在隔离 venv），失败自动降级 yt-dlp
- **不要预检字幕轨**：实测 `has_subs()` 预检与实际下载结果不一致，会误杀能抓到的视频。直接抓，失败就跳到下一条
- 间隔 3 秒即可（比 yt-dlp 路径安全得多）

---

### ❌ 已知无效：Bo Burnham 官方频道

诊断结果为 `TranscriptsDisabled`（**内容方主动关闭字幕**），不是工具问题。
他的视频（如 `1Rx_p3NW7gQ`、`ObOqq1knVxs`、`k1BneeJTDcU`、`xHotXbGZiFY`）全部无字幕轨。
**同类风险**：部分版权方/官方频道会主动关字幕，遇错就换人，不要反复重试同一频道。

### ✅ 喜剧/脱口秀推荐（字幕覆盖好，实测可抓）

| 人物 | 搜索词 | 实测结果 |
|---|---|---|
| **George Carlin** | `george carlin money` / `life is precious` | 4/4 成功，最长 33624 字 |
| **Richard Feynman** | `richard feyndnan cargo cult` | 2/2 成功，17576 / 21359 字 |
| **Chris Rock** | `chris rock human race` | 2/3 成功，16k 字符左右 |
| ~~Bo Burnham~~ | — | 字幕被官方关闭，不可用 |

喜剧方向的价值在于**"把真话说得好笑的形态"**——学结构，不学段子。

---

### ❌ 不要凭记忆猜 video ID

实测：凭记忆猜的 ID 大批量失效（unavailable / truncated / 429）。**这是浪费轮次的主要来源。**
即使 `youtube-transcript-api` 报 `VideoUnavailable`，也要先确认这个 ID 是不是编的。
真实流程一律走`yt_search.py <关键词>` 或 WebSearch。

### 还该覆盖的方向（下次优先）
- **关系与沟通**：Chris Graydon、Ariel Goldstein、关系心理学
- **创业决策**：真实创业者访谈（Stripe、Airbnb 创始人访谈）
- **习惯与改变**：James Clear、Wendell Berry
- **金钱观**：George Carlin 的 money 专场已入库，可直接提炼

---


## 二、工作流程

### A. 学习模式（持续进行，不需要用户触发）

1. **找素材**：TED 用 `ted_transcript.py --list`；YouTube 用 `yt_search.py <关键词>`
2. **抓全文**：TED 走官方 transcript；YouTube 走 transcript-api（间隔 3 秒）
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

全部脚本在工作区根目录：`C:\Users\wenmin.xiong\WorkBuddy\2026-10-10-09-56-03\`

| 脚本 | 用途 | 用法 |
|---|---|---|
| `ted_transcript.py` | TED 官方逐字稿（首选） | `python ted_transcript.py --list <关键词> 5`<br>`python ted_transcript.py <slug>` |
| `yt_search.py` | YouTube 搜索 + 抓字幕 | `python yt_search.py <关键词>`<br>`python yt_search.py --grab <关键词> --n 3` |
| `yt_sub.py` | 单条字幕（降级备用） | `python yt_sub.py <video_id> en` |

Python 解释器（含 `youtube-transcript-api`）：
```
C:\Users\wenmin.xiong\.workbuddy\binaries\python\envs\default\Scripts\python.exe
```

### 抓取铁律（都是实测踩过的坑）
- ❌ 凭记忆猜 video ID
- ❌ TED transcript URL 加 `?language=` 参数（会破坏解析）
- ❌ 用带完整 Chrome UA 抓 TED（403 Forbidden-UA），用简短 UA
- ❌ 依赖 `has_subs()` 预检来决定跳过（与实际结果不一致，会误杀）
- ✅ TED 抓取直连，不限流，可连续抓
- ✅ YouTube 抓取间隔 3 秒，单次可抓多条
- ✅ 遇 `TranscriptsDisabled` 直接换人，那是内容方关字幕

### 素材库位置
- 字幕原文：`subs/`
  - `subs/ted_<slug>.en.txt` — TED 官方稿
  - `subs/<video_id>.<lang>.txt` — YouTube 字幕
- 观点提炼与转化文案：工作区根目录各 HTML 文件

---

## 五、已抓演讲清单

### YouTube 字幕

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
| `3AO2U16uoeI` | Tim Urban 停止拖延 | 394 段 |
| `Rk5C149J9C0` | 拖延与行动 | 短稿 |

### TED 官方稿（`subs/ted_*.txt`，质量最高，优先读）

| slug | 主题 | 字数 |
|---|---|---|
| `sir_ken_robinson_do_schools_kill_creativity` | 创造力/教育 | 17678 / 105段 |
| `ayelet_fishbach_4_proven_ways_to_kick_your_procrastination_habit` | 拖延习惯 | 23395 / 49段 |
| `jon_youshaei_is_perfectionism_just_procrastination_in_disguise` | 完美主义=拖延 | 5884 |
| `ted_ed_why_you_procrastinate_even_when_it_feels_bad` | 拖延机制 | 4508 |

### 喜剧/脱口秀（学"把真话说得好笑的形态"）

| ID | 人物 | 主题 | 字数 |
|---|---|---|---|
| `3Qlf0oCpWgQ` | George Carlin | money | **33624**（最长稿） |
| `7N7njl6Ncnw` | George Carlin | 金钱观 | 3112 |
| `AY95vuEjTTE` | George Carlin | Life Is Sacred | — |
| `6MVABINRzN4` | George Carlin | A Modern Man | — |
| `rH4YRYSOuKk` | George Carlin | Life Is Worth Losing（全长） | — |
| `RtR_ECkqblg` | Richard Feynman | Cargo Cult | **17576** |
| `HpE-Us8OZoM` | Richard Feynman | Cargo Cult Speech | **21359** |
| `nEJP40vQ1xI` | Chris Rock | Human Race | 16419 |
| `8xMwYKwXYvA` | Chris Rock | Human Race | 16294 |

> **Feynman Cargo Cult 是金矿**：讲人做什么不思考就跟着做、把自己降级成cargo（货物）、
> 嘴上说一套做一套却以为自己在解决问题——几乎可以直译成短视频共鸣文案。

---

## 六、数据反推（发满 10 条后执行）

用户发满 10 条后，要播放量最高和最低各 3 条，用真实数据反推账号该吃哪类观点、哪种情绪浓度、多长的句子。

**数据比任何规律都准。**

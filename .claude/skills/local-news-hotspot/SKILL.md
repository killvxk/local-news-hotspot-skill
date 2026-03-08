---
name: local-news-hotspot
description: 采集并分析 worldmonitor 项目的实时新闻数据源，生成即时热点新闻，并根据用户地理位置显示本地相关的重要消息。当用户询问"热点新闻"、"最新新闻"、"有什么新闻"、"本地新闻"或任何关于当前事件的请求时使用此技能。
---

# 本地热点新闻分析器

此技能从 worldmonitor 项目已识别的多个权威数据源采集实时新闻，分析热点话题，并展示最重要的新闻，特别强调与用户地理位置相关的本地新闻。

## 核心工作流程

当此技能被调用时，按以下步骤执行：

### 1. 检测用户位置

首先，确定用户的地理位置以提供本地相关新闻：

```bash
curl -s https://ipinfo.io/json
```

从响应中提取：
- `city`: 用户所在城市
- `region`: 省份/州
- `country`: 国家代码和名称
- `loc`: 纬度和经度

如果 API 调用失败，默认只显示全球新闻，不进行本地过滤。

### 2. 采集新闻数据

基于 worldmonitor 项目分析的数据源结构，从多个高可信度来源获取新闻。优先使用 Tier 1 和 Tier 2 来源：

**Tier 1 来源（通讯社 - 最高优先级）**：
- Reuters World: `https://news.google.com/rss/search?q=site:reuters.com+world&hl=en-US&gl=US&ceid=US:en`
- AP News: `https://news.google.com/rss/search?q=site:apnews.com&hl=en-US&gl=US&ceid=US:en`
- BBC World: `https://feeds.bbci.co.uk/news/world/rss.xml`
- Bloomberg Markets: `https://news.google.com/rss/search?q=site:bloomberg.com+markets+when:1d&hl=en-US&gl=US&ceid=US:en`

**Tier 2 来源（主要媒体）**：
- Al Jazeera: `https://www.aljazeera.com/xml/rss/all.xml`
- CNN World: `https://news.google.com/rss/search?q=site:cnn.com+world+news+when:1d&hl=en-US&gl=US&ceid=US:en`
- Guardian World: `https://www.theguardian.com/world/rss`
- NPR News: `https://feeds.npr.org/1001/rss.xml`

**区域来源**（根据检测到的位置选择）：
- **亚洲**：
  - BBC Asia: `https://feeds.bbci.co.uk/news/world/asia/rss.xml`
  - NHK World: `https://news.google.com/rss/search?q=site:nhk.or.jp+when:1d&hl=en-US&gl=US&ceid=US:en`
  - Nikkei Asia: `https://news.google.com/rss/search?q=site:asia.nikkei.com+when:3d&hl=en-US&gl=US&ceid=US:en`
  - The Diplomat: `https://thediplomat.com/feed/`
- **欧洲**：
  - EuroNews: `https://www.euronews.com/rss?format=xml`
  - France 24: `https://www.france24.com/en/rss`
  - DW News: `https://rss.dw.com/xml/rss-en-all`
- **美洲**：
  - NPR News: `https://feeds.npr.org/1001/rss.xml`
  - CBS News: `https://www.cbsnews.com/latest/rss/main`
  - Guardian Americas: `https://www.theguardian.com/world/americas/rss`
- **中东**：
  - Al Jazeera: `https://www.aljazeera.com/xml/rss/all.xml`
  - BBC Middle East: `https://feeds.bbci.co.uk/news/world/middle_east/rss.xml`
- **非洲**：
  - BBC Africa: `https://feeds.bbci.co.uk/news/world/africa/rss.xml`
- **中国**：
  - Xinhua: `https://news.google.com/rss/search?q=site:xinhuanet.com+OR+Xinhua+when:1d&hl=en-US&gl=US&ceid=US:en`

**重要说明**：
- 大部分来源使用 **Google News RSS 聚合**（格式：`https://news.google.com/rss/search?q=site:...`）
- 这是 worldmonitor 项目验证过的策略，优势：
  - 绕过某些网站的 RSS 访问限制
  - 统一的 XML 格式，更易解析
  - 更好的可用性和稳定性
- 少数来源使用直接 RSS（BBC、Guardian、NPR、Al Jazeera 等）

使用以下命令获取 RSS feeds：
```bash
curl -s -L -A "Mozilla/5.0 (compatible; NewsBot/1.0)" --max-time 10 "<feed-url>"
```

从 XML 中提取：
- 标题 (title)
- 描述/摘要 (description/summary)
- 发布日期 (pubDate/published)
- 链接 (link)
- 分类/标签 (category/tags，如果可用)

### 3. 分析和排名新闻

应用以下评分系统对新闻项目进行排名：

**时效性得分** (0-40 分)：
- 最近 1 小时：40 分
- 1-3 小时：30 分
- 3-6 小时：20 分
- 6-12 小时：10 分
- 12-24 小时：5 分
- 更早：0 分

**来源可信度** (0-30 分)：
- Tier 1 (通讯社)：30 分
- Tier 2 (主要媒体)：20 分
- Tier 3 (专业来源)：10 分

**本地相关性** (0-30 分)：
- 提及用户所在城市：30 分
- 提及用户所在省份/州：25 分
- 提及用户所在国家：20 分
- 提及邻国：10 分
- 无本地关联：0 分

**话题重要性** (0-20 分)：
根据关键词匹配分配分数：
- 突发/紧急/警报：20 分
- 冲突/战争/袭击/灾难：18 分
- 政治危机/选举：15 分
- 经济/市场崩盘：15 分
- 重大政策变化：12 分
- 技术突破：10 分
- 常规新闻：5 分

**去重检测**：
- 如果多个来源报道同一新闻（标题相似度 >70%），只保留得分最高的版本
- 将重复信息合并到单个条目中，注明"报道来源：[来源1], [来源2]"

### 4. 格式化输出

以清晰、终端友好的格式呈现结果：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 您的位置: [城市], [省份], [国家]
🕐 生成时间: [本地时区的当前时间戳]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔥 全球热点新闻 (3-5 条)

[每条新闻:]
[排名]. [表情符号] [标题]
   📰 来源: [来源名称] | ⏰ [多久之前]
   📍 地点: [如果提及地理位置]
   💬 [简短的 1-2 句摘要]
   🔗 [URL]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📍 [用户位置] 本地新闻 (如果有，显示 3-5 条)

[格式同上，但过滤本地相关性]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**表情符号选择指南**：
- 🚨 突发/紧急新闻
- ⚔️ 冲突/军事
- 🏛️ 政治/政府
- 💰 经济/金融
- 🌍 国际关系
- 🔬 科学/技术
- ⚠️ 灾难/紧急情况
- 📊 市场/商业
- 🗳️ 选举
- 🌡️ 气候/环境

### 5. 处理边缘情况

**未找到本地新闻**：
如果找到少于 3 条本地新闻，显示：
```
📍 [位置] 本地新闻
   ℹ️  过去 24 小时内没有重大本地新闻。
   改为显示 [更广泛区域] 的区域新闻。
```

**API 失败**：
- 如果位置检测失败：只显示全球新闻
- 如果 RSS feeds 失败：尝试备用来源，显示部分结果
- 如果所有来源都失败：显示错误消息和故障排除步骤

**速率限制**：
- 缓存结果 15 分钟以避免过多 API 调用
- 将缓存存储在 `/tmp/news-hotspot-cache-[timestamp].json`

## 实现说明

**性能**：
- 使用后台进程并行获取 RSS feeds
- 每个 feed 设置 10 秒超时
- 每个 feed 最多处理 50 个项目

**语言支持**：
- 从位置（国家代码）检测用户语言
- 如果用户在中国/台湾：包含中文来源（新华社、CCTV）
- 如果用户在日本：包含 NHK、日经
- 如果用户在韩国：包含 Yonhap、KBS
- 如果用户在阿拉伯语国家：包含 Al Jazeera Arabic、Al Arabiya

**隐私**：
- 位置检测通过公共 IP 地理定位完成
- 不存储或传输个人数据
- 所有数据在本地处理

## 使用示例

**用户提示**: "给我看看最新热点新闻"

**预期输出**:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 您的位置: 上海, 上海, 中国
🕐 生成时间: 2026-03-08 16:45:23 CST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔥 全球热点新闻

1. 🚨 突发：太平洋地区发生重大地震
   📰 来源: Reuters | ⏰ 23 分钟前
   📍 地点: 太平洋，日本附近
   💬 7.2 级地震袭击太平洋，多国发布海啸警报。
   🔗 https://reuters.com/...

2. ⚔️ 中东冲突局势升级
   📰 来源: BBC World | ⏰ 1 小时前
   📍 地点: 加沙地带
   💬 国际调解员呼吁立即停火，人道主义危机加深。
   🔗 https://bbc.com/...

[... 3 条更多新闻 ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📍 上海, 中国 本地新闻

1. 💰 上海股市创新高
   📰 来源: 新华社 | ⏰ 2 小时前
   📍 地点: 上海, 中国
   💬 上海综合指数在科技板块强劲表现下收于历史高位。
   🔗 https://xinhua.com/...

[... 2-4 条更多本地新闻 ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 错误处理

如果遇到错误：

1. **网络问题**：使用指数退避重试 (1秒, 2秒, 4秒)
2. **XML 解析错误**：跳过格式错误的 feeds，继续处理其他
3. **空结果**：将时间窗口扩展到 48 小时
4. **位置检测失败**：仅继续全球新闻

始终提供部分结果而不是完全失败。如果只找到 1-2 条新闻，显示它们并注明结果有限。

## 成功标准

成功执行应该：
- 在 30 秒内完成
- 返回 3-5 条全球热点
- 返回 3-5 条本地新闻（如果可用）
- 显示清晰、可读的终端输出
- 包含所有新闻的有效 URL
- 显示准确的时间戳
- 正确检测用户位置

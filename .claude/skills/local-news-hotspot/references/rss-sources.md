# RSS 数据源配置

本文档包含所有新闻数据源的完整 RSS URLs，基于 worldmonitor 项目验证。

## Tier 1 来源（通讯社）

最高优先级，最可靠的新闻来源：

| 来源 | URL | 说明 |
|------|-----|------|
| **Reuters World** | `https://news.google.com/rss/search?q=site:reuters.com+world&hl=en-US&gl=US&ceid=US:en` | Google News 聚合 |
| **AP News** | `https://news.google.com/rss/search?q=site:apnews.com&hl=en-US&gl=US&ceid=US:en` | Google News 聚合 |
| **BBC World** | `https://feeds.bbci.co.uk/news/world/rss.xml` | 直接 RSS |
| **Bloomberg Markets** | `https://news.google.com/rss/search?q=site:bloomberg.com+markets+when:1d&hl=en-US&gl=US&ceid=US:en` | Google News 聚合，限制 1 天内 |

## Tier 2 来源（主要媒体）

高质量新闻来源：

| 来源 | URL | 说明 |
|------|-----|------|
| **Al Jazeera** | `https://www.aljazeera.com/xml/rss/all.xml` | 直接 RSS |
| **CNN World** | `https://news.google.com/rss/search?q=site:cnn.com+world+news+when:1d&hl=en-US&gl=US&ceid=US:en` | Google News 聚合，限制 1 天内 |
| **Guardian World** | `https://www.theguardian.com/world/rss` | 直接 RSS |
| **NPR News** | `https://feeds.npr.org/1001/rss.xml` | 直接 RSS |

## 区域来源

根据用户位置选择相应的区域来源。

### 亚洲

| 来源 | URL | 说明 |
|------|-----|------|
| **BBC Asia** | `https://feeds.bbci.co.uk/news/world/asia/rss.xml` | 直接 RSS |
| **NHK World** | `https://news.google.com/rss/search?q=site:nhk.or.jp+when:1d&hl=en-US&gl=US&ceid=US:en` | Google News 聚合，1 天内 |
| **Nikkei Asia** | `https://news.google.com/rss/search?q=site:asia.nikkei.com+when:3d&hl=en-US&gl=US&ceid=US:en` | Google News 聚合，3 天内 |
| **The Diplomat** | `https://thediplomat.com/feed/` | 直接 RSS |

### 欧洲

| 来源 | URL | 说明 |
|------|-----|------|
| **EuroNews** | `https://www.euronews.com/rss?format=xml` | 直接 RSS |
| **France 24** | `https://www.france24.com/en/rss` | 直接 RSS |
| **DW News** | `https://rss.dw.com/xml/rss-en-all` | 直接 RSS |

### 美洲

| 来源 | URL | 说明 |
|------|-----|------|
| **NPR News** | `https://feeds.npr.org/1001/rss.xml` | 直接 RSS |
| **CBS News** | `https://www.cbsnews.com/latest/rss/main` | 直接 RSS |
| **Guardian Americas** | `https://www.theguardian.com/world/americas/rss` | 直接 RSS |

### 中东

| 来源 | URL | 说明 |
|------|-----|------|
| **Al Jazeera** | `https://www.aljazeera.com/xml/rss/all.xml` | 直接 RSS |
| **BBC Middle East** | `https://feeds.bbci.co.uk/news/world/middle_east/rss.xml` | 直接 RSS |

### 非洲

| 来源 | URL | 说明 |
|------|-----|------|
| **BBC Africa** | `https://feeds.bbci.co.uk/news/world/africa/rss.xml` | 直接 RSS |

### 中国

| 来源 | URL | 说明 |
|------|-----|------|
| **Xinhua** | `https://news.google.com/rss/search?q=site:xinhuanet.com+OR+Xinhua+when:1d&hl=en-US&gl=US&ceid=US:en` | Google News 聚合，1 天内 |

## Google News 聚合说明

大部分来源使用 Google News RSS 聚合策略：

**格式**：
```
https://news.google.com/rss/search?q=site:DOMAIN+KEYWORDS&hl=LANG&gl=COUNTRY&ceid=COUNTRY:LANG
```

**参数**：
- `q`: 搜索查询（`site:` 限制域名）
- `when:Nd`: 限制 N 天内的新闻
- `hl`: 语言代码
- `gl`: 国家代码
- `ceid`: 国家和语言组合

**优势**：
- 绕过某些网站的 RSS 访问限制
- 统一的 XML 格式，更易解析
- 更好的可用性和稳定性
- 可以添加时间过滤

## 获取 RSS 命令

```bash
curl -s -L -A "Mozilla/5.0 (compatible; NewsBot/1.0)" --max-time 10 "<feed-url>"
```

**参数说明**：
- `-s`: 静默模式
- `-L`: 跟随重定向
- `-A`: 设置 User-Agent
- `--max-time 10`: 10 秒超时

## XML 字段提取

从 RSS/Atom feeds 中提取以下字段：

| 字段 | RSS 标签 | Atom 标签 |
|------|----------|-----------|
| 标题 | `<title>` | `<title>` |
| 描述 | `<description>` | `<summary>` |
| 发布日期 | `<pubDate>` | `<published>` 或 `<updated>` |
| 链接 | `<link>` | `<link href="...">` |
| 分类 | `<category>` | `<category term="...">` |

## 来源分级

| Tier | 类型 | 评分 | 示例 |
|------|------|------|------|
| **Tier 1** | 通讯社 | 30 分 | Reuters, AP, BBC |
| **Tier 2** | 主要媒体 | 20 分 | CNN, Guardian, NPR |
| **Tier 3** | 专业来源 | 10 分 | The Diplomat, Nikkei |

## 区域映射

根据用户国家代码选择区域来源：

| 国家代码 | 区域 | 来源 |
|----------|------|------|
| CN, TW, HK | 亚洲 | BBC Asia, NHK, Nikkei, Xinhua |
| JP | 亚洲 | BBC Asia, NHK, Nikkei |
| KR | 亚洲 | BBC Asia, NHK, Nikkei |
| SG, MY, TH, VN, PH, ID | 亚洲 | BBC Asia, The Diplomat |
| GB, FR, DE, IT, ES, NL, BE, SE, NO, DK | 欧洲 | EuroNews, France 24, DW News |
| US, CA, MX, BR, AR | 美洲 | NPR, CBS, Guardian Americas |
| SA, AE, QA, KW, BH, OM, IQ, IR, IL, JO, LB, SY, YE | 中东 | Al Jazeera, BBC Middle East |
| EG, ZA, NG, KE, ET, GH, TZ, UG | 非洲 | BBC Africa |

## 性能优化

- **并行获取**：使用后台进程同时获取多个 feeds
- **超时设置**：每个 feed 10 秒超时
- **项目限制**：每个 feed 最多处理 50 个项目
- **缓存策略**：结果缓存 15 分钟

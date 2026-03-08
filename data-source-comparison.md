# 数据源对比分析：技能 vs worldmonitor

## 概述

本文档对比分析了 `local-news-hotspot` 技能使用的数据源与 worldmonitor 项目实际使用的数据源。

---

## 关键发现

### ❌ 数据源**不完全一致**

技能中定义的 RSS URLs 与 worldmonitor 实际使用的 URLs **存在显著差异**。

---

## 详细对比

### 1. Reuters (路透社)

| 项目 | URL |
|------|-----|
| **技能定义** | `https://feeds.reuters.com/reuters/worldNews` |
| **worldmonitor 实际** | `https://news.google.com/rss/search?q=site:reuters.com+world&hl=en-US&gl=US&ceid=US:en` |
| **差异** | ⚠️ 技能使用直接 RSS，worldmonitor 通过 Google News 聚合 |

### 2. AP News (美联社)

| 项目 | URL |
|------|-----|
| **技能定义** | `https://feeds.apnews.com/rss/apf-topnews` |
| **worldmonitor 实际** | `https://news.google.com/rss/search?q=site:apnews.com&hl=en-US&gl=US&ceid=US:en` |
| **差异** | ⚠️ 技能使用直接 RSS，worldmonitor 通过 Google News 聚合 |

### 3. BBC World

| 项目 | URL |
|------|-----|
| **技能定义** | `https://feeds.bbci.co.uk/news/world/rss.xml` |
| **worldmonitor 实际** | `https://feeds.bbci.co.uk/news/world/rss.xml` |
| **差异** | ✅ **完全一致** |

### 4. Bloomberg

| 项目 | URL |
|------|-----|
| **技能定义** | `https://feeds.bloomberg.com/markets/news.rss` |
| **worldmonitor 实际** | ❌ **未找到** Bloomberg RSS feed |
| **差异** | ⚠️ 技能包含但 worldmonitor 可能不使用此源 |

### 5. Al Jazeera

| 项目 | URL |
|------|-----|
| **技能定义** | `https://www.aljazeera.com/xml/rss/all.xml` |
| **worldmonitor 实际** | 需要进一步查找（可能在其他分类中） |
| **差异** | ⚠️ 需要验证 |

### 6. CNN World

| 项目 | URL |
|------|-----|
| **技能定义** | `http://rss.cnn.com/rss/cnn_world.rss` |
| **worldmonitor 实际** | `https://news.google.com/rss/search?q=site:cnn.com+world+news+when:1d&hl=en-US&gl=US&ceid=US:en` |
| **差异** | ⚠️ 技能使用直接 RSS，worldmonitor 通过 Google News 聚合（且限制 1 天内） |

### 7. Guardian World

| 项目 | URL |
|------|-----|
| **技能定义** | `https://www.theguardian.com/world/rss` |
| **worldmonitor 实际** | `https://www.theguardian.com/world/rss` |
| **差异** | ✅ **完全一致** |

### 8. NPR News

| 项目 | URL |
|------|-----|
| **技能定义** | `https://feeds.npr.org/1001/rss.xml` |
| **worldmonitor 实际** | `https://feeds.npr.org/1001/rss.xml` |
| **差异** | ✅ **完全一致** |

---

## 核心差异分析

### 1. Google News 聚合 vs 直接 RSS

**worldmonitor 的策略**：
- 对于 Reuters、AP、CNN 等主要来源，使用 Google News RSS 聚合
- 格式：`https://news.google.com/rss/search?q=site:reuters.com+world&...`
- **优势**：
  - 绕过某些网站的 RSS 限制
  - 统一的格式和结构
  - 更好的可用性（Google News 作为中间层）
- **劣势**：
  - 依赖 Google News 服务
  - 可能有延迟
  - 受 Google 的内容过滤影响

**技能的策略**：
- 直接使用官方 RSS feeds
- **优势**：
  - 更直接，无中间层
  - 更快的更新速度
  - 不依赖第三方聚合服务
- **劣势**：
  - 某些 RSS 可能被限制或失效
  - 格式不统一，需要更复杂的解析

### 2. 数据源覆盖范围

**worldmonitor**：
- 435+ RSS feeds
- 覆盖 15 个类别
- 包含多语言支持（21 种语言）
- 包含区域特定来源（亚洲、欧洲、中东、非洲等）

**技能当前定义**：
- 仅定义了 8 个核心来源（Tier 1 和 Tier 2）
- 主要是英语来源
- 区域来源仅在文档中提及，未提供具体 URLs

### 3. 来源分级系统

**worldmonitor**：
```typescript
SOURCE_TIERS = {
  'Reuters': 1,      // Wire services
  'AP News': 1,
  'BBC World': 2,    // Major outlets
  'Guardian World': 2,
  'Defense One': 3,  // Specialty
  'Hacker News': 4,  // Aggregators
}
```

**技能**：
```markdown
Tier 1: Reuters, AP, BBC, Bloomberg (30分)
Tier 2: Al Jazeera, CNN, Guardian, NPR (20分)
Tier 3: Specialty sources (10分)
```

✅ **分级系统基本一致**，但技能简化了分类

---

## 实际测试中使用的数据源

根据测试输出，代理实际采集的来源：

### 测试 1 (eval-1-latest-hotspots)
- BBC World ✅
- The Guardian ✅
- Al Jazeera ✅
- CNN ✅
- BBC Asia ⚠️ (技能文档中提到但未明确定义 URL)

### 测试 2 (eval-2-important-news)
- BBC World ✅
- BBC Asia ⚠️
- CNN World ✅
- Al Jazeera ✅

### 测试 3 (eval-3-local-news)
- BBC World ✅
- Al Jazeera ✅
- CNN World ✅
- Channel NewsAsia ⚠️ (未在技能中定义)
- The Straits Times ⚠️ (未在技能中定义)

**观察**：代理在执行时**自行添加了额外的区域来源**（Channel NewsAsia、The Straits Times），这些并未在技能文档中明确定义。

---

## 问题与建议

### 问题

1. **URL 不一致**：技能定义的 RSS URLs 与 worldmonitor 实际使用的不同
2. **覆盖范围有限**：技能只定义了 8 个核心来源，而 worldmonitor 有 435+
3. **缺少具体 URLs**：技能文档中提到区域来源，但没有提供具体的 RSS URLs
4. **代理自由发挥**：测试中代理添加了未定义的来源，说明技能指导不够明确

### 建议

#### 选项 1：完全对齐 worldmonitor
- 从 worldmonitor 的 `feeds.ts` 提取完整的 RSS URLs
- 使用相同的 Google News 聚合策略
- 复制完整的 435+ 数据源列表

**优势**：
- 与成熟项目保持一致
- 利用已验证的数据源
- 更全面的新闻覆盖

**劣势**：
- 技能文件会非常大
- 维护复杂度高
- 可能超出技能的简洁性要求

#### 选项 2：精选核心来源 + 明确 URLs
- 保持当前的精简策略（8-15 个核心来源）
- 但使用 worldmonitor 验证过的 URLs
- 为每个区域明确定义 2-3 个来源

**优势**：
- 保持技能简洁
- 使用可靠的 URLs
- 更容易维护

**劣势**：
- 覆盖范围有限
- 可能错过某些重要新闻

#### 选项 3：引用 worldmonitor 配置文件
- 技能直接读取 worldmonitor 的 `feeds.ts`
- 动态加载数据源配置
- 保持与 worldmonitor 同步

**优势**：
- 自动同步更新
- 完整的数据源覆盖
- 无需重复维护

**劣势**：
- 依赖 worldmonitor 项目存在
- 技能不再独立
- 需要解析 TypeScript 配置

---

## 结论

**当前状态**：技能的数据源定义与 worldmonitor **不完全一致**。

**主要差异**：
1. worldmonitor 使用 Google News 聚合，技能使用直接 RSS
2. worldmonitor 有 435+ 来源，技能只定义了 8 个
3. 某些 URLs 不同（Reuters、AP、CNN）
4. 技能缺少具体的区域来源 URLs

**建议行动**：
- 更新技能以使用 worldmonitor 验证过的 URLs
- 为每个区域明确定义 2-3 个核心来源
- 考虑使用 Google News 聚合策略以提高可靠性
- 或者直接引用 worldmonitor 的配置文件

---

**生成时间**: 2026-03-08
**分析基于**:
- `/root/news/.claude/skills/local-news-hotspot/SKILL.md`
- `/root/news/examples/worldmonitor/src/config/feeds.ts`
- 测试输出文件

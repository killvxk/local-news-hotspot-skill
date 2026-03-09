# 技能更新日志

## 版本 1.2 - 2026-03-08

### 🎨 重构：遵循 skill-development 最佳实践

**变更内容**：

#### 结构优化（Progressive Disclosure）
- ✅ **SKILL.md 精简**：从 267 行（~3,500 字）减少到 ~1,800 字
- ✅ **创建 references/ 目录**：
  - `references/rss-sources.md` - 完整的 RSS 数据源配置（150 行）
  - `references/scoring-algorithm.md` - 详细的评分算法文档（300+ 行）
- ✅ **渐进式加载**：核心流程在 SKILL.md，详细内容在 references/

#### 描述改进
- ✅ **第三人称格式**：
  - 旧: "采集并分析 worldmonitor 项目的实时新闻数据源..."
  - 新: "This skill should be used when the user asks to..."
- ✅ **具体触发短语**：添加了 "show hot news", "latest news", "breaking news" 等明确触发词

#### 写作风格
- ✅ **祈使句/不定式**：全文使用命令式语气，避免第二人称
- ✅ **清晰的资源引用**：明确指向 references/ 文件

#### 版本号
- 从 1.1 升级到 1.2

### 📝 文档结构

**新的文件组织**：
```
.claude/skills/local-news-hotspot/
├── SKILL.md (精简版，1,800 字)
├── README.md
├── CHANGELOG.md
├── evals/
│   └── evals.json
└── references/
    ├── rss-sources.md (完整数据源配置)
    └── scoring-algorithm.md (详细评分算法)
```

### 🎯 影响

**优势**:
1. ✅ 符合 skill-development 最佳实践
2. ✅ 更好的可维护性（关注点分离）
3. ✅ 更快的加载速度（SKILL.md 更小）
4. ✅ 更准确的触发（第三人称 + 具体短语）
5. ✅ 更好的可扩展性（详细内容在 references/）

**对比 v1.1**:
- SKILL.md 大小：267 行 → ~120 行（减少 55%）
- 触发准确性：提升（添加了具体触发短语）
- 文档组织：单文件 → 多文件分层结构

---

## 版本 1.1 - 2026-03-08

### 🔄 数据源更新：对齐 worldmonitor 实际 URLs

**变更内容**：

#### Tier 1 来源（通讯社）
- ✅ **Reuters World**:
  - 旧: `https://feeds.reuters.com/reuters/worldNews`
  - 新: `https://news.google.com/rss/search?q=site:reuters.com+world&hl=en-US&gl=US&ceid=US:en`
  - 原因: 使用 Google News 聚合，提高可用性

- ✅ **AP News**:
  - 旧: `https://feeds.apnews.com/rss/apf-topnews`
  - 新: `https://news.google.com/rss/search?q=site:apnews.com&hl=en-US&gl=US&ceid=US:en`
  - 原因: 使用 Google News 聚合

- ✅ **Bloomberg**:
  - 旧: `https://feeds.bloomberg.com/markets/news.rss`
  - 新: `https://news.google.com/rss/search?q=site:bloomberg.com+markets+when:1d&hl=en-US&gl=US&ceid=US:en`
  - 原因: 使用 Google News 聚合，限制 1 天内新闻

- ✅ **BBC World**: 保持不变
  - URL: `https://feeds.bbci.co.uk/news/world/rss.xml`

#### Tier 2 来源（主要媒体）
- ✅ **CNN World**:
  - 旧: `http://rss.cnn.com/rss/cnn_world.rss`
  - 新: `https://news.google.com/rss/search?q=site:cnn.com+world+news+when:1d&hl=en-US&gl=US&ceid=US:en`
  - 原因: 使用 Google News 聚合，限制 1 天内新闻

- ✅ **Al Jazeera**: 保持不变
  - URL: `https://www.aljazeera.com/xml/rss/all.xml`

- ✅ **Guardian World**: 保持不变
  - URL: `https://www.theguardian.com/world/rss`

- ✅ **NPR News**: 保持不变
  - URL: `https://feeds.npr.org/1001/rss.xml`

#### 区域来源（新增具体 URLs）
之前只有名称，现在添加了完整的 RSS URLs：

**亚洲**:
- BBC Asia: `https://feeds.bbci.co.uk/news/world/asia/rss.xml`
- NHK World: `https://news.google.com/rss/search?q=site:nhk.or.jp+when:1d&hl=en-US&gl=US&ceid=US:en`
- Nikkei Asia: `https://news.google.com/rss/search?q=site:asia.nikkei.com+when:3d&hl=en-US&gl=US&ceid=US:en`
- The Diplomat: `https://thediplomat.com/feed/`

**欧洲**:
- EuroNews: `https://www.euronews.com/rss?format=xml`
- France 24: `https://www.france24.com/en/rss`
- DW News: `https://rss.dw.com/xml/rss-en-all`

**美洲**:
- NPR News: `https://feeds.npr.org/1001/rss.xml`
- CBS News: `https://www.cbsnews.com/latest/rss/main`
- Guardian Americas: `https://www.theguardian.com/world/americas/rss`

**中东**:
- Al Jazeera: `https://www.aljazeera.com/xml/rss/all.xml`
- BBC Middle East: `https://feeds.bbci.co.uk/news/world/middle_east/rss.xml`

**非洲**:
- BBC Africa: `https://feeds.bbci.co.uk/news/world/africa/rss.xml`

**中国**:
- Xinhua: `https://news.google.com/rss/search?q=site:xinhuanet.com+OR+Xinhua+when:1d&hl=en-US&gl=US&ceid=US:en`

### 📝 文档更新

- 添加了 Google News 聚合策略说明
- 更新了 README.md 中的技术实现部分
- 创建了数据源对比分析文档（`/root/news/data-source-comparison.md`）

### 🎯 影响

**优势**:
1. ✅ 与 worldmonitor 项目保持一致
2. ✅ 使用经过验证的可靠 URLs
3. ✅ Google News 聚合提高了可用性
4. ✅ 区域来源现在有具体的 URLs，不再模糊

**注意事项**:
- Google News URLs 较长，但更稳定
- 某些来源限制了时间范围（1天或3天）
- 需要测试新 URLs 的实际表现

### 🔜 下一步

- [ ] 重新运行测试用例验证新 URLs
- [ ] 对比新旧 URLs 的性能和质量
- [ ] 根据测试结果进一步优化

---

## 版本 1.0 - 2026-03-08

初始版本，基于 worldmonitor 项目分析创建。

**功能**:
- 自动检测用户地理位置
- 从多个权威 RSS 源采集新闻
- 四维度评分系统（时效性、可信度、本地相关性、重要性）
- 终端友好的格式化输出
- 全球热点 + 本地新闻双重展示

**数据源**: 8 个核心来源 + 区域来源（但 URLs 不完全准确）

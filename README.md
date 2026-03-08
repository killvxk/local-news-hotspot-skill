# Local News Hotspot Skill

一个基于 worldmonitor 项目数据源的本地热点新闻聚合技能。

## 功能特性

- 🌍 自动检测用户地理位置
- 📰 从多个权威 RSS 源采集实时新闻
- 🎯 智能评分和排名（时效性、可信度、本地相关性、重要性）
- 📍 显示 3-5 条全球热点 + 3-5 条本地相关新闻
- 💻 终端友好的格式化输出

## 数据源

使用 worldmonitor 项目验证过的 RSS URLs：

- **Tier 1 来源**（通讯社）：Reuters, AP, BBC, Bloomberg
- **Tier 2 来源**（主要媒体）：Al Jazeera, CNN, Guardian, NPR
- **区域来源**：亚洲、欧洲、美洲、中东、非洲、中国特定来源

主要采用 Google News RSS 聚合策略，提高可用性和稳定性。

## 技能位置

```
.claude/skills/local-news-hotspot/
├── SKILL.md          # 技能主文件
├── README.md         # 技能说明
├── CHANGELOG.md      # 版本日志
└── evals/
    └── evals.json    # 测试用例
```

## 使用方法

在 Claude Code 中，当用户询问以下内容时会自动触发：

- "热点新闻"
- "最新新闻"
- "有什么新闻"
- "本地新闻"
- "breaking news"
- "what's happening"

或者直接调用：
```bash
/local-news-hotspot
```

## 输出示例

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 您的位置: Shanghai, Shanghai, China
🕐 生成时间: 2026-03-08 16:45:23 CST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔥 全球热点新闻

1. 🚨 BREAKING: Major earthquake strikes Pacific region
   📰 来源: Reuters | ⏰ 23 分钟前
   📍 Location: Pacific Ocean, near Japan
   💬 A magnitude 7.2 earthquake has struck...
   🔗 https://reuters.com/...

[... 更多新闻 ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📍 Shanghai, China 本地新闻

1. 💰 Shanghai stock market reaches new high
   📰 来源: Xinhua | ⏰ 2 小时前
   📍 Location: Shanghai, China
   💬 Shanghai Composite Index closes at record levels...
   🔗 https://xinhua.com/...

[... 更多本地新闻 ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 技术实现

- **位置检测**: ipinfo.io API
- **数据采集**: RSS feeds + Google News 聚合
- **评分算法**: 四维度加权（时效性 40% + 可信度 30% + 本地相关性 30% + 重要性 20%）
- **缓存策略**: 15 分钟缓存
- **性能目标**: < 30 秒

## 版本历史

- **v1.1** (2026-03-08): 数据源对齐 worldmonitor 实际 URLs
- **v1.0** (2026-03-08): 初始版本

详见 [CHANGELOG.md](.claude/skills/local-news-hotspot/CHANGELOG.md)

## 相关文档

- [数据源对比分析](data-source-comparison.md)
- [技能更新总结](skill-update-summary.md)

## 基于项目

本技能基于 [worldmonitor](https://github.com/koala73/worldmonitor) 项目的数据源分析创建。

## 许可证

MIT License

## 作者

Created with Claude (Opus 4.6)

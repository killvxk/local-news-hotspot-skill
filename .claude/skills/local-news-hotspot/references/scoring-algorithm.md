# 新闻评分算法详解

本文档详细说明 local-news-hotspot 技能使用的多维度新闻评分系统。

## 评分维度

新闻总分 = 时效性得分 + 来源可信度 + 本地相关性 + 话题重要性

最高总分：120 分

---

## 1. 时效性得分 (0-40 分)

时效性是最重要的维度，占总分的 33%。新闻越新鲜，得分越高。

| 发布时间 | 得分 | 说明 |
|---------|------|------|
| 最近 1 小时 | 40 分 | 突发新闻，最高优先级 |
| 1-3 小时 | 30 分 | 非常新鲜 |
| 3-6 小时 | 20 分 | 较新 |
| 6-12 小时 | 10 分 | 当天新闻 |
| 12-24 小时 | 5 分 | 昨天新闻 |
| 超过 24 小时 | 0 分 | 过时新闻 |

**计算方法**：

```bash
# 获取新闻发布时间戳
pub_timestamp=$(date -d "$pub_date" +%s)
current_timestamp=$(date +%s)
age_seconds=$((current_timestamp - pub_timestamp))
age_hours=$((age_seconds / 3600))

# 计算时效性得分
if [ $age_hours -le 1 ]; then
  recency_score=40
elif [ $age_hours -le 3 ]; then
  recency_score=30
elif [ $age_hours -le 6 ]; then
  recency_score=20
elif [ $age_hours -le 12 ]; then
  recency_score=10
elif [ $age_hours -le 24 ]; then
  recency_score=5
else
  recency_score=0
fi
```

---

## 2. 来源可信度 (0-30 分)

基于 worldmonitor 项目的来源分级系统，占总分的 25%。

| Tier | 类型 | 得分 | 示例 |
|------|------|------|------|
| **Tier 1** | 通讯社 | 30 分 | Reuters, AP, BBC, Bloomberg |
| **Tier 2** | 主要媒体 | 20 分 | CNN, Guardian, NPR, Al Jazeera |
| **Tier 3** | 专业来源 | 10 分 | The Diplomat, Nikkei Asia, EuroNews |

**来源映射表**：

```bash
# Tier 1 来源（30 分）
tier1_sources=(
  "Reuters"
  "AP News"
  "Associated Press"
  "BBC"
  "Bloomberg"
)

# Tier 2 来源（20 分）
tier2_sources=(
  "CNN"
  "Guardian"
  "NPR"
  "Al Jazeera"
  "France 24"
  "DW News"
  "CBS News"
)

# Tier 3 来源（10 分）
tier3_sources=(
  "The Diplomat"
  "Nikkei Asia"
  "EuroNews"
  "NHK World"
  "Xinhua"
)
```

---

## 3. 本地相关性 (0-30 分)

根据新闻内容与用户位置的关联程度评分，占总分的 25%。

| 关联级别 | 得分 | 匹配条件 |
|---------|------|---------|
| 城市级 | 30 分 | 标题或描述提及用户所在城市 |
| 省份/州级 | 25 分 | 提及用户所在省份/州 |
| 国家级 | 20 分 | 提及用户所在国家 |
| 邻国级 | 10 分 | 提及邻国或同一区域 |
| 无关联 | 0 分 | 无地理关联 |

**匹配逻辑**：

```bash
# 从 ipinfo.io 获取用户位置
user_city="Shanghai"
user_region="Shanghai"
user_country="China"

# 检查新闻内容
title_and_desc="$title $description"

# 城市匹配（30 分）
if echo "$title_and_desc" | grep -qi "$user_city"; then
  local_score=30
# 省份匹配（25 分）
elif echo "$title_and_desc" | grep -qi "$user_region"; then
  local_score=25
# 国家匹配（20 分）
elif echo "$title_and_desc" | grep -qi "$user_country"; then
  local_score=20
# 邻国匹配（10 分）
elif echo "$title_and_desc" | grep -Eqi "Japan|Korea|Vietnam|Taiwan"; then
  local_score=10
else
  local_score=0
fi
```

**邻国映射表**：

| 用户国家 | 邻国列表 |
|---------|---------|
| China | Japan, Korea, Vietnam, Taiwan, Mongolia, Russia, India |
| USA | Canada, Mexico |
| UK | Ireland, France, Belgium, Netherlands |
| Germany | France, Poland, Austria, Switzerland, Netherlands, Belgium |
| Japan | China, Korea, Russia |

---

## 4. 话题重要性 (0-20 分)

基于关键词匹配判断新闻的重要程度，占总分的 17%。

| 话题类型 | 得分 | 关键词 |
|---------|------|--------|
| 突发/紧急 | 20 分 | breaking, urgent, alert, emergency, just in |
| 冲突/灾难 | 18 分 | war, conflict, attack, disaster, earthquake, tsunami, hurricane |
| 政治危机 | 15 分 | crisis, election, coup, impeachment, scandal |
| 经济重大 | 15 分 | crash, collapse, recession, bailout, default |
| 重大政策 | 12 分 | policy, reform, law, regulation, treaty |
| 技术突破 | 10 分 | breakthrough, innovation, discovery, launch |
| 常规新闻 | 5 分 | 其他 |

**关键词匹配逻辑**：

```bash
title_lower=$(echo "$title" | tr '[:upper:]' '[:lower:]')
desc_lower=$(echo "$description" | tr '[:upper:]' '[:lower:]')
content="$title_lower $desc_lower"

# 突发/紧急（20 分）
if echo "$content" | grep -Eq "breaking|urgent|alert|emergency|just in"; then
  importance_score=20
# 冲突/灾难（18 分）
elif echo "$content" | grep -Eq "war|conflict|attack|disaster|earthquake|tsunami|hurricane|flood|fire"; then
  importance_score=18
# 政治危机（15 分）
elif echo "$content" | grep -Eq "crisis|election|coup|impeachment|scandal|protest|riot"; then
  importance_score=15
# 经济重大（15 分）
elif echo "$content" | grep -Eq "crash|collapse|recession|bailout|default|bankruptcy"; then
  importance_score=15
# 重大政策（12 分）
elif echo "$content" | grep -Eq "policy|reform|law|regulation|treaty|agreement|deal"; then
  importance_score=12
# 技术突破（10 分）
elif echo "$content" | grep -Eq "breakthrough|innovation|discovery|launch|unveil"; then
  importance_score=10
# 常规新闻（5 分）
else
  importance_score=5
fi
```

---

## 5. 去重检测

如果多个来源报道同一新闻，只保留得分最高的版本。

**相似度计算**：

使用简单的词汇重叠率：

```bash
# 提取标题关键词（去除停用词）
title1_words=$(echo "$title1" | tr '[:upper:]' '[:lower:]' | grep -oE '\w+' | grep -vE '^(the|a|an|in|on|at|to|for|of|and|or|but|is|are|was|were)$')
title2_words=$(echo "$title2" | tr '[:upper:]' '[:lower:]' | grep -oE '\w+' | grep -vE '^(the|a|an|in|on|at|to|for|of|and|or|but|is|are|was|were)$')

# 计算交集和并集
common_words=$(comm -12 <(echo "$title1_words" | sort) <(echo "$title2_words" | sort) | wc -l)
total_words=$(echo "$title1_words $title2_words" | tr ' ' '\n' | sort -u | wc -l)

# 相似度 = 交集 / 并集
similarity=$(echo "scale=2; $common_words / $total_words" | bc)

# 如果相似度 > 0.7，视为重复
if (( $(echo "$similarity > 0.7" | bc -l) )); then
  echo "Duplicate detected"
fi
```

**去重策略**：

1. 对所有新闻按总分排序
2. 从高分到低分遍历
3. 对每条新闻，检查是否与已选新闻相似
4. 如果相似度 > 70%，跳过该新闻
5. 否则，加入结果集

**合并信息**：

如果检测到重复，合并来源信息：

```
标题: Major earthquake strikes Pacific region
来源: Reuters, BBC, CNN (3 个来源报道)
```

---

## 6. 排名和筛选

**全球热点新闻**：

1. 计算所有新闻的总分
2. 按总分降序排序
3. 应用去重检测
4. 选择前 3-5 条（总分 ≥ 60 分）

**本地新闻**：

1. 筛选本地相关性得分 > 0 的新闻
2. 按总分降序排序
3. 应用去重检测
4. 选择前 3-5 条（总分 ≥ 40 分）

**最低分数阈值**：

- 全球热点：60 分（确保高质量）
- 本地新闻：40 分（放宽标准以显示更多本地内容）

---

## 7. 示例计算

**示例 1：突发地震新闻**

```
标题: "Breaking: 7.2 magnitude earthquake strikes near Tokyo"
来源: Reuters (Tier 1)
发布时间: 15 分钟前
地点: Tokyo, Japan
用户位置: Shanghai, China

计算：
- 时效性：15 分钟前 → 40 分
- 来源可信度：Reuters (Tier 1) → 30 分
- 本地相关性：提及邻国 Japan → 10 分
- 话题重要性：breaking + earthquake → 20 分

总分 = 40 + 30 + 10 + 20 = 100 分
```

**示例 2：本地经济新闻**

```
标题: "Shanghai stock market reaches record high"
来源: Xinhua (Tier 3)
发布时间: 2 小时前
地点: Shanghai, China
用户位置: Shanghai, China

计算：
- 时效性：2 小时前 → 30 分
- 来源可信度：Xinhua (Tier 3) → 10 分
- 本地相关性：提及城市 Shanghai → 30 分
- 话题重要性：常规经济新闻 → 5 分

总分 = 30 + 10 + 30 + 5 = 75 分
```

**示例 3：常规国际新闻**

```
标题: "European leaders meet to discuss climate policy"
来源: Guardian (Tier 2)
发布时间: 8 小时前
地点: Brussels, Belgium
用户位置: Shanghai, China

计算：
- 时效性：8 小时前 → 10 分
- 来源可信度：Guardian (Tier 2) → 20 分
- 本地相关性：无关联 → 0 分
- 话题重要性：policy → 12 分

总分 = 10 + 20 + 0 + 12 = 42 分
```

---

## 8. 调优建议

**如果全球热点太少**：

- 降低最低分数阈值（60 → 50）
- 扩展时间窗口（24 小时 → 48 小时）

**如果本地新闻太少**：

- 降低最低分数阈值（40 → 30）
- 放宽地理匹配（包含更多邻国）
- 扩展到区域新闻（如"亚洲新闻"）

**如果结果太多**：

- 提高最低分数阈值
- 收紧去重检测（70% → 60%）
- 限制每个来源的最大新闻数

---

## 9. 性能优化

**并行评分**：

```bash
# 使用后台进程并行计算每条新闻的得分
for news_item in "${news_items[@]}"; do
  (
    score=$(calculate_score "$news_item")
    echo "$score|$news_item"
  ) &
done
wait

# 收集结果并排序
sort -t'|' -k1 -rn scored_news.txt
```

**缓存策略**：

- 缓存位置检测结果（15 分钟）
- 缓存 RSS feeds（15 分钟）
- 缓存评分结果（15 分钟）

**批量处理**：

- 每个 feed 最多处理 50 个项目
- 总共最多处理 500 个新闻项目
- 评分后立即筛选，减少内存占用

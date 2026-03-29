---
name: news
description: 查询市场消息和新闻，包括 X/Twitter、财经网站等外部消息源。适用场景："今天为什么涨/跌？" "NVDA 有什么消息？" "市场发生了什么？"
---

# /news — 市场消息查询

**目的**：查询外部消息源，获取市场或个股相关的实时新闻、传闻、分析师观点。

---

## 查询方式

根据用户问题类型，选择合适的信息源：

### 1. 整体市场消息（Why is the market up/down today?）

**信息源优先级**：
1. **TradingView News** — 实时市场新闻聚合
2. **Yahoo Finance** — 头条新闻
3. **X/Twitter 搜索** — `$SPY`, `$QQQ`, `#stocks` 等标签

**执行步骤**：
```python
# 获取今日主要指数涨跌
import yfinance as yf
for symbol in ['SPY', 'QQQ', 'VIX']:
    ticker = yf.Ticker(symbol)
    hist = ticker.history(period='2d')
    # 计算涨跌幅

# 尝试抓取 Yahoo Finance 新闻 headlines
# 尝试抓取 TradingView 新闻
```

### 2. 个股消息（SYMBOL 有什么新闻？）

**信息源**：
1. **Yahoo Finance Ticker News** — `https://finance.yahoo.com/quote/{SYMBOL}/news/`
2. **Seeking Alpha** — 分析师文章
3. **X/Twitter** — `${SYMBOL}`, `#SYMBOL`

**执行步骤**：
```python
# 1. 获取股票今日表现
# 2. 用 WebFetch 抓取 Yahoo Finance 新闻页面
# 3. 提取新闻标题和摘要
```

### 3. X/Twitter 特定查询

由于 X API 需要开发者账号，skill 采用以下策略：
- **优先**：使用 WebFetch 抓取 X 的公开搜索页面（如 `https://x.com/search?q=$NVDA&f=live`）
- **备选**：指导用户如何自行查看

---

## 输出格式

```markdown
## 市场消息速报 · [日期时间]

### 今日市场表现
- SPY: +2.01%
- QQQ: +2.03%
- NVDA: +2.87% (用户持仓)

### 头条新闻
1. [标题] — [来源]
   [摘要]

2. [标题] — [来源]
   [摘要]

### X/Twitter 热议
- [推文摘要或趋势话题]

### 分析师观点
- [Seeking Alpha / 其他来源的分析师观点]

### 总结
[一句话总结今天涨跌的主要原因]
```

---

## 使用示例

**用户输入**：
- `/news` — 查询整体市场今天的主要消息
- `/news NVDA` — 查询 NVDA 相关新闻
- `/news twitter` — 特别强调查询 X/Twitter 消息

---

## 技术实现

**依赖**：
- `yfinance` — 获取价格数据
- `WebFetch` — 抓取新闻页面
- `WebSearch` — 搜索引擎备用

**限制说明**：
- X/Twitter 公开页面抓取可能受限于反爬虫机制
- 实时性取决于信息源的更新频率
- 某些网站（如 Bloomberg、WSJ）需要订阅，无法直接抓取

---

## 备选方案

如果 WebFetch 无法获取有效内容，向用户提供以下自助查询建议：

1. **TradingView**：打开 tradingview.com → 搜索 SPY → 右侧 "News" 面板
2. **X/Twitter 搜索**：
   - `$SPY` — 标普500相关讨论
   - `$NVDA` — NVDA相关讨论
   - `#stocks` — 股票话题
   - `from:DeItaone` — 知名财经推主
3. **财联社/华尔街见闻**：中文实时快讯

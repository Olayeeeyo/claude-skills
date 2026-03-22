---
name: scan
description: 扫描关注股票，寻找当前有机会的标的。当用户输入 /scan 时触发。适用场景："有什么机会？" "现在哪只股票值得关注？"
---

# /scan — 机会扫描

**目的**：快速浏览所有关注标的的当前状态，找出值得进一步研究的候选标的。

## 执行步骤

### Step 1：获取评分概览

```bash
cd D:\AI_about\invest_analysis
python tui_query.py overview
```

读取输出，记录每只股票的综合评分和信号方向。

### Step 2：获取市场情绪

```bash
python tui_query.py sentiment
```

关注：
- VIX 水平（< 18 低恐慌，18-25 中性，> 25 高恐慌）
- SPY/QQQ 趋势（多头 / 空头 / 震荡）
- 当前市场环境适合进攻还是防守

### Step 3：筛选候选标的

**排除**：IAU、BND、QQQ（长期持有 ETF，不做择时）

对评分前三的标的，快速检查戴维斯双击条件：
```bash
python tui_query.py valuation SYMBOL
```

戴维斯双击信号 = EPS 超预期（forward_eps > trailing_eps）AND PE 低于历史 50 百分位

### Step 4：输出报告

按以下格式输出：

```
## 市场环境
[一句话总结：VIX 水平 + 趋势偏向]

## 评分排行（前5，排除 ETF）
1. SYMBOL — 评分 X/100，信号：买入/卖出/中性
2. ...

## 重点候选
### SYMBOL
- 当前评分：X
- 戴维斯条件：✓ 满足 / ✗ 不满足（说明原因）
- 建议下一步：/deep SYMBOL 获取完整分析

## 结论
[1-2 句话：当前适合操作还是观望，哪只标的最值得关注]
```

## 注意事项

- 扫描是粗筛，不是买入信号
- 市场情绪差（VIX > 25）时，即使信号强也要谨慎
- 发现候选后用 `/deep SYMBOL` 做深度分析

---
name: portfolio
description: 投资分析总入口，当问题不明确属于哪类分析时触发。适用场景："今天情况怎么样？" "给我完整报告" "全面分析" "持仓怎么样" "有什么需要关注的"。也适用于单股提问（自动组合 deep+timing）。明确指定 /scan /risk /deep /timing 时直接用对应 skill。
---

# /portfolio — 投资分析中枢

**目的**：根据用户问题自动路由到正确的分析 skill，或并行运行全部 4 个 skill 生成完整日报。

---

## 意图路由

根据用户的问题，判断应该调用哪个 skill：

| 用户问题类型 | 调用 | 说明 |
|-------------|------|------|
| "有什么机会" / "该关注什么" / "扫描" | `scan` | 单独调用 |
| "有什么风险" / "期权风险" / "持仓情况" | `risk` | 单独调用 |
| "[SYMBOL] 怎么样" / "值得买吗" / "分析一下 X" | `deep SYMBOL` + `timing SYMBOL` | 同时调用，输出整合报告 |
| "X 现在能买吗" / "时机对吗" | `timing SYMBOL` | 单独调用 |
| "X 基本面如何" / "X 的估值" | `deep SYMBOL` | 单独调用 |
| "完整报告" / "今天情况" / "全面分析" / "4 个 skill 都跑" | 全部 4 个 | 见下方并行执行步骤 |

---

## 完整日报：并行执行全部 4 个 Skill

当用户要求完整报告时，使用 `superpowers:dispatching-parallel-agents` 模式，同时启动 4 个后台 agent：

**Agent 1 — scan-agent**
> 调用 `scan` skill，返回机会扫描报告

**Agent 2 — risk-agent**
> 调用 `risk` skill，返回持仓风险报告

**Agent 3 — deep-agent**
> 依次对 config.json 中 stocks 列表调用 `deep` skill

**Agent 4 — timing-agent**
> 依次对同一列表调用 `timing` skill

等全部 agent 完成后，汇总为一份完整报告，结构：
1. 市场环境（来自 scan）
2. 持仓风险（来自 risk）
3. 个股分析（deep + timing 整合，按股票分节）
4. 今日行动清单

---

## 单股整合分析（deep + timing）

对单只股票提问时，同时运行 `deep` 和 `timing`，合并输出：

```
## [SYMBOL] 分析报告
### 基本面与估值（来自 deep）
[估值、EPS、分析师评级、营收预期]

### 技术面与择时（来自 timing）
[信号共振、关键价位、入场建议]

### 综合结论
[一句话：现在是否值得操作，以及具体理由]
```

---

## 快速参考

| Skill | 回答什么问题 | 需要指定股票 |
|-------|------------|------------|
| `/scan` | 现在哪只股票值得关注？ | 否 |
| `/deep SYMBOL` | 这只股票基本面/估值怎么样？ | 是 |
| `/timing SYMBOL` | 现在是买入/卖出的好时机吗？ | 是 |
| `/risk` | 我的持仓有什么风险？ | 否 |

**活跃分析标的**：从 `config.json` 的 `stocks` 字段读取

---

## 注意事项

- 问题明确时（如直接输入 `/scan`），无需经过本 skill，直接调用对应 skill
- 完整日报并行运行时间较长（约 2-3 分钟），正常现象
- 关注股票列表在 `config.json` 的 `stocks` 字段，如有变动需同步更新本 skill

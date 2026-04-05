---
name: wiki-reflect
description: 将个人记录与 Wiki 知识对比反思。用法：/wiki-reflect "反思问题" 或 /wiki-reflect --month 2026-03。自动发现知行不一的地方。
---

# /wiki-reflect

将个人记录（投资日志）与 Wiki 知识对比，发现知行不一的地方。

## 用法

```
/wiki-reflect "反思问题"
/wiki-reflect --month YYYY-MM
/wiki-reflect --recent N  # 最近 N 天，默认 30
```

示例：
- `/wiki-reflect "我的交易和学过的知识矛盾吗？"`
- `/wiki-reflect --month 2026-03`
- `/wiki-reflect --recent 14`

## 执行流程

### Step 0: 读取规则

首先读取 WIKI-SCHEMA.md 获取所有规则。

### Step 1: 读取个人记录（Haiku 子进程）

```
Agent(model="haiku", prompt="
你是一个个人记录分析器。阅读以下投资日志，提取关键决策和观点。

时间范围：[用户指定的时间范围，默认最近 30 天]

投资日志内容：
[Note/投资管理/ 目录下相关文件的完整内容]

任务：
1. 提取关键交易决策（买入、卖出、加仓、减仓等）
2. 提取明确表达的观点（"我觉得市场..."、"我认为..."等）
3. 识别相关的 Wiki 标签（期权、仓位管理、风险对冲等）

输出严格遵循 WIKI-SCHEMA.md 第 9.4 节的 JSON 格式。
")
```

### Step 2: 匹配 Wiki 知识（主进程 Sonnet）

根据 Step 1 提取的 `relevant_wiki_tags`，读取相关 Wiki 页面：

```markdown
相关 Wiki 页面：
- wiki/concepts/仓位管理.md
- wiki/concepts/市场预测.md
- wiki/topics/期权风险管理.md
...
```

### Step 3: 对比分析（主进程 Sonnet）

将个人决策与 Wiki 知识对比，输出反思报告：

```markdown
# 反思报告：2026年3月

分析范围：8 篇投资日志，12 条交易决策

---

## 一致（3 处）

1. **3/12 买入保护性 put**
   - 符合"风险对冲优先"原则
   - 参考：[[wiki/topics/期权风险管理]]

2. **仓位控制在合理范围**
   - 符合"单标的不超过 10%"规则
   - 参考：[[wiki/concepts/仓位管理]]

3. **未追涨**
   - 与"逆向思维"原则一致

---

## 矛盾（2 处）

1. **3/25 加仓 NVDA put**
   - 你在日志中写道："恐慌中加仓"
   - 加仓后 NVDA 占比达 15%
   - 但你总结的规则是"单标的不超过 10%"（[[wiki/concepts/仓位管理]]）
   - ⚠️ 你突破了自定的规则

2. **3/28 预测市场**
   - 你在日志中写道："觉得市场已经见底"
   - 但你读《随机漫步的傻瓜》总结的是"不要预测市场方向"（[[wiki/concepts/市场预测]]）
   - ⚠️ 知道但没做到

---

## 知识缺口（3 处）

以下决策在 Wiki 中没有找到对应的决策框架：
- 3/18 "分批止盈"
- 3/22 "看到支撑位"
- 3/29 "情绪化减仓"

建议消化相关资料补充这些知识。

---

> 保存为 wiki/synthesis/交易反思-2026年3月.md？[y/n]
```

### Step 4: 留存判断（主进程）

使用 AskUserQuestion 询问是否保存为 synthesis 页面。

如果用户选择 y：
1. 在 wiki/synthesis/ 下创建反思报告
2. 更新 wiki/index.md
3. 追加 wiki/log.md

## 输出格式

反思报告包含三部分：

1. **一致**：行为符合 Wiki 知识
2. **矛盾**：行为违反 Wiki 知识（⚠️ 标记）
3. **知识缺口**：有行为但 Wiki 中无对应知识

## 特殊情况

### 个人记录为空

如果指定时间范围内没有投资日志：

```markdown
未找到时间范围内的投资日志。

当前 JOURNAL 层：Note/投资管理/
检查该目录下是否有文件。
```

### Wiki 中无相关知识

如果 Wiki 中没有与个人记录相关的知识：

```markdown
Wiki 中暂无与你的交易相关的知识。

建议：
1. 先使用 /wiki-ingest 消化 RAW/ 目录下的投资相关书籍和文章
2. 建立基础知识框架后再进行反思
```

## 错误处理

- Note/投资管理/ 目录不存在 → 报告"JOURNAL 层未找到"
- Haiku 返回非 JSON → 报错并终止
- Wiki 页面读取失败 → 跳过该页面，标注"知识暂不可用"
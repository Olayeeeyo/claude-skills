---
name: wiki-query
description: 基于 Wiki 知识库回答问题。用法：/wiki-query "你的问题"。自动检索相关 Wiki 页面，综合回答并附引用。回答可保存为新的 synthesis 页面。
---

# /wiki-query

基于 Wiki 知识库回答问题，回答附带来源引用。

## 用法

```
/wiki-query "你的问题"
```

示例：
- `/wiki-query "期权卖方策略有哪些？"`
- `/wiki-query "安全边际的概念是什么？"`
- `/wiki-query "我的交易日志和学过的知识有矛盾吗？"`（涉及个人记录）

## 执行流程

### Step 0: 读取规则

首先读取 WIKI-SCHEMA.md 获取所有规则。

### Step 1: 检索（Haiku 子进程）

```
Agent(model="haiku", prompt="
你是一个 Wiki 检索器。根据用户问题和 Wiki 索引，找出最相关的页面。

用户问题：[用户的问题]

Wiki 索引：
[wiki/index.md 的内容]

任务：
1. 找出 5-10 个最相关的 Wiki 页面
2. 判断问题是否涉及个人记录（检测是否提到"我的""我""日志"等指向个人的词）
3. 如果涉及个人记录，设置 involves_journal: true

输出严格遵循 WIKI-SCHEMA.md 第 9.3 节的 JSON 格式。
")
```

### Step 2: 综合回答（主进程 Sonnet）

1. 读取 Step 1 找到的 Wiki 页面
2. 如果 `involves_journal: true`，同时读取 JOURNAL 层相关文件
3. 基于读取的内容综合回答
4. 回答中附带 `[[wiki/...]]` 格式的引用

回答格式：

```markdown
**回答**：[直接回答问题，2-3 段]

**相关 Wiki 页面**：
- [[wiki/concepts/安全边际]] — 概念说明
- [[wiki/topics/价值投资方法论]] — 专题说明

**来源溯源**：
> 来自《投资最重要的事》：[引用原文]
```

### Step 3: 留存判断（主进程）

如果回答涉及跨页面综合分析（检索了 3 个以上 Wiki 页面），询问用户是否保存：

```
这个回答涉及跨页面综合分析。要保存为新的 Wiki 页面吗？
[y/n]
```

如果用户选择 y：
1. 在 wiki/synthesis/ 下创建新页面
2. 页面标题根据问题生成
3. 更新 wiki/index.md
4. 追加 wiki/log.md

## 输出格式

```markdown
**回答**：

期权卖方策略主要包括：

1. **卖出看跌期权** — 适合愿意在低价接盘的情况...
   参考：[[wiki/concepts/卖出看跌期权]]

2. **备兑看涨期权** — 适合已持有正股的情况...
   参考：[[wiki/concepts/备兑看涨期权]]

**相关 Wiki 页面**：
- [[wiki/concepts/卖出看跌期权]]
- [[wiki/concepts/备兑看涨期权]]
- [[wiki/topics/期权卖方策略]]

> 保存为 synthesis 页面？[y/n]
```

## 特殊情况

### 问题涉及个人记录

如果 Haiku 判断问题涉及个人记录，回答中会包含：

```markdown
> [!note] 来自个人记录
> 你在 Note/投资管理/ 的日志中提到...，这与 Wiki 中的 XX 概念一致/矛盾。
```

### Wiki 中无相关知识

如果检索结果为空（relevant_pages 为空数组）：

```markdown
抱歉，Wiki 中还没有与"XXX"相关的知识。

建议：
1. 使用 /wiki-ingest 消化 RAW/ 目录下的相关资料
2. 或者告诉我你想了解什么，我可以帮你找到相关来源
```

## 错误处理

- Haiku 返回非 JSON → 报错并终止
- Wiki 页面不存在（index 引用但文件缺失）→ 报告问题，跳过该页面
- JOURNAL 文件读取失败 → 仅基于 Wiki 回答，标注"个人记录暂不可用"
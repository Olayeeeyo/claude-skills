---
name: wiki-ingest
description: 消化一个 RAW 文件，更新 Wiki 知识库。用法：/wiki-ingest Clippings/文章名.md。自动提取标签、概念、实体，创建或更新相关 Wiki 页面。
---

# /wiki-ingest

消化一个 RAW 层文件，将知识整合进 Wiki 系统。

## 用法：

```
/wiki-ingest <文件路径>
```

示例：
- `/wiki-ingest RAW/Clippings/covered-call策略详解.md`
- `/wiki-ingest RAW/读书笔记/07-价值投资/投资最重要的事.md`

## 执行流程

### Step 0: 读取规则

首先读取 WIKI-SCHEMA.md 获取所有规则。

### Step 1: 预检查（本地）

1. 检查文件是否存在
2. 计算 文件 md5 hash
3. 检查 wiki/log.md 中是否已有该文件的记录
   - 如果有记录且 hash 相同 → 文件未修改，跳过处理
   - 如果有记录但 hash 不同 → 文件已修改，需要重新处理
   - 如果无记录 → 新文件，需要处理

### Step 2: 分类 + 标签提取（Haiku 子进程）

使用 Agent 工具派遣 Haiku 子进程：

```
Agent(model="haiku", prompt="
你是一个文档分类器。阅读以下文件内容和 WIKI-SCHEMA.md 的规则，输出 JSON。

文件内容：
[文件完整内容]

任务：
1. 判断文件类型：raw（知识来源）或 journal（个人记录）
2. 提取标签（3-8 个关键词）
3. 提取实体（人名、书名、公司名等）
4. 提取概念（术语、方法论、理论等）
5. 提取核心观点（3-5 条）

输出严格遵循 WIKI-SCHEMA.md 第 9.1 节的 JSON 格式。
")
```

如果 Haiku 返回 `file_type: journal`，终止处理并告知用户"该文件属于个人记录，不作为知识源处理"。

### Step 3: 关联发现（Haiku 子进程）

```
Agent(model="haiku", prompt="
你是一个 Wiki 关联分析器。根据以下信息，找出需要创建或更新的 Wiki 页面。

已提取的信息：
[Step 2 的 JSON 输出]

当前 Wiki 索引：
[wiki/index.md 的内容]

任务：
1. 找出需要新建的 Wiki 页面（概念/实体/主题）
2. 找出需要更新的已有 Wiki 页面
3. 检测可能的冲突（新观点与已有 Wiki 内容矛盾）

输出严格遵循 WIKI-SCHEMA.md 第 9.2 节的 JSON 格式。
")
```

### Step 4: Wiki 写入（主进程 Sonnet）

根据 Step 2 和 Step 3 的结果，执行 Wiki 页面的创建/更新：

1. **新建页面**：
   - 在正确的子目录下创建文件（concepts/、entities/、topics/）
   - 按 WIKI-SCHEMA.md 第 3 节的格式写入
   - frontmatter 的 sources 包含当前处理的文件路径

2. **更新页面**：
   - 读取已有页面内容
   - 追加新的 `### 来自《来源》` 子节
   - 如果存在冲突，添加 `> [!warning]` callout
   - 更新 frontmatter 的 updated、sources、related 字段

3. **维护交叉引用**：
   - 更新 related 字段，确保双向链接
   - 更新 wiki/index.md

### Step 5: 日志记录（主进程）

追加记录到 wiki/log.md：

```markdown
## [YYYY-MM-DD] ingest | 文件标题

- 源: 文件路径
- 标签: tag1, tag2, tag3
- 新建: concepts/xxx.md, entities/yyy.md
- 更新: topics/zzz.md
- 冲突: (如有)
- hash: [md5]
```

## 输出格式

处理完成后，向用户报告：

```
✅ Wiki 更新完成

源文件: RAW/Clippings/covered-call策略详解.md
标签: 期权, covered call, 卖方策略

新建页面:
- wiki/concepts/备兑看涨期权.md

更新页面:
- wiki/topics/期权卖方策略.md (+820 字)

可在 Obsidian 中查看变化。
```

## 错误处理

- 文件不存在 → 报错并终止
- Haiku 返回非 JSON → 报错并终止，不写入任何文件
- 文件属于 JOURNAL 层 → 告知用户，不作为知识源处理
- 写入过程中断 → log.md 最后写入，下次可检测并恢复
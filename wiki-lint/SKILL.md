---
name: wiki-lint
description: Wiki 健康检查。用法：/wiki-lint。检查孤页、缺页、冲突、未消化文件等问题，输出结构化报告。
---

# /wiki-lint

检查 Wiki 系统的健康状态，发现并报告问题。

## 用法

```
/wiki-lint
```

## 执行流程

### Step 0: 读取规则

首先读取 WIKI-SCHEMA.md 获取所有规则。

### Step 1: 结构检查（Haiku 子进程）

```
Agent(model="haiku", prompt="
你是一个 Wiki 结构检查器。检查以下内容并输出问题列表。

Wiki 索引：
[wiki/index.md 的内容]

Wiki 目录结构：
[wiki/ 下所有文件的路径列表]

操作日志：
[wiki/log.md 的内容]

RAW 层文件列表：
[RAW/ 目录下所有文件的列表，带 md5 hash]

检查项目：
1. 孤页：没有任何 inbound link 的页面（检查 index.md 中哪些页面从未被其他页面的 related 字段引用）
2. 缺页：被引用但不存在的 [[link]]
3. 陈旧：源文件 hash 与 log.md 记录不一致（源文件已修改但 Wiki 未更新）
4. 未消化：RAW/ 目录下文件未出现在 log.md 中

输出 JSON 格式：
{
  "orphan_pages": ["wiki/concepts/xxx.md", ...],
  "missing_pages": [{"from": "wiki/topics/aaa.md", "link": "wiki/concepts/bbb.md"}, ...],
  "stale_pages": ["wiki/concepts/ccc.md", ...],
  "undigested_files": ["RAW/Clippings/xxx.md", ...],
  "total_pages": N,
  "total_links": N
}
")
```

### Step 2: 语义检查（主进程 Sonnet，仅在 Step 1 发现问题时执行）

如果 Step 1 发现问题，读取相关页面进行语义分析：

```
1. 矛盾检测：
   - 检查 confidence: contested 的页面
   - 列出具体矛盾内容

2. 知识缺口：
   - 分析 tags 分布
   - 找出明显缺失的知识领域
```

### Step 3: 报告输出（主进程）

输出结构化报告：

```markdown
# Wiki 健康检查报告

**统计**：47 个页面, 189 条交叉引用

## 结构问题

### 孤页（2 个）
- wiki/entities/瑞达利欧.md — 0 条入链
- wiki/concepts/均值回归.md — 0 条入链

### 缺页（1 个）
- wiki/concepts/凯利公式.md — 被 3 个页面引用但不存在

### 陈旧（3 个）
- wiki/concepts/Theta衰减.md — 源文件已修改

### 未消化（14 个）
- RAW/Clippings/xxx.md
- RAW/Clippings/yyy.md
...

## 语义问题

### 矛盾（1 处）
- wiki/concepts/Theta衰减.md：30天加速 vs 45天加速

### 知识缺口
- 期权知识丰富（23 页），但风险管理只有 4 页
- 建议补充：止损策略、组合对冲

---
报告已记录到 wiki/log.md
```

### Step 4: 日志记录

追加记录到 wiki/log.md：

```markdown
## [YYYY-MM-DD] lint

- 总页面: N
- 总链接: N
- 孤页: N 个
- 缺页: N 个
- 陈旧: N 个
- 未消化: N 个文件
- 矛盾: N 处
```

## 输出格式

直接在对话中输出报告，不使用 AskUserQuestion（lint 是只读操作，不需要用户决策）。

## 错误处理

- wiki/index.md 不存在 → 报告"Wiki 系统未初始化"
- wiki/log.md 不存在 → 创建空文件
- Haiku 返回非 JSON → 报告"结构检查失败"，跳过 Step 2
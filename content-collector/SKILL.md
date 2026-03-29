---
name: content-collector
description: >-
  Content collector that processes files from ~/Downloads or CLI arguments
  (images, documents, URLs) and organizes them into an Obsidian vault.
  Use when user wants to collect, import, or save content into Obsidian.
  Trigger keywords: /content-collector, collect content, 内容收集, 整理到 Obsidian.
---

# Content Collector - 内容收集器

从 ~/Downloads 文件夹或命令行参数接收输入（图片/文档/网址），提取内容并整理到 Obsidian 文档库。

## 核心原则

1. **优先使用 Obsidian CLI** 进行文档操作
2. **处理前读取 CLAUDE.md**，确保归类符合文档库结构
3. **海明威写作原则**：精简表达，但不遗漏信息
4. **自动删除原文件**，保持 Downloads 文件夹整洁

## 支持的输入类型

| 类型 | 扩展名/格式 | 处理方式 |
|------|-------------|----------|
| 图片 | jpg, jpeg, png, heic, webp | 视觉提取内容 |
| 文本文档 | txt, md | 直接读取内容 |
| 网址 | http(s):// | 调用 `/markdown-proxy` 获取内容 |

## 使用方法

```bash
# 自动扫描并处理 Downloads 文件夹中的所有支持的文件
/content-collector

# 处理特定文件
/content-collector ~/Downloads/file.txt
/content-collector ~/Downloads/image.png

# 处理网址
/content-collector https://example.com/article
```

## 处理流程

### 1. 读取 CLAUDE.md 了解文档库结构

目标文件夹结构：
- `读书笔记/` - 读书/学习/知识类
- `期权教程/` - 期权交易相关
- `投资管理/` - 投资策略/理财
- `内容收集/` - 杂项/无法归类
- `旅游攻略/` - 旅行相关

### 2. 判断输入类型并处理

**图片文件**：
- 使用视觉能力查看图片
- 提取文字和信息
- 应用海明威原则精简

**文本文档**：
- 直接读取文件内容
- 应用海明威原则精简

**网址**：
- 调用 `/markdown-proxy` skill 获取 markdown 格式内容
- 应用海明威原则精简

### 3. 智能归类规则

根据内容关键词自动判断目标文件夹：

| 关键词 | 目标文件夹 |
|--------|-----------|
| 旅游/旅行/酒店/景点/攻略 | `旅游攻略/` |
| 期权/期权交易/option | `期权教程/` |
| 股票/投资/理财/基金/ETF | `投资管理/` |
| AI/模型/LLM/技术/编程 | `AI 工具/` 或 `技术学习/` |
| 读书/书摘/笔记/学习 | `读书笔记/` |
| 其他 | `内容收集/` |

### 4. Obsidian CLI 操作

```bash
# 分析文档库结构，获取现有文件夹
obsidian eval code="app.vault.getFiles().map(f => f.path).join('\n')"

# 创建文档到子文件夹
obsidian eval code="(async () => {
  const content = \`---
title: 文档标题
type: note
created: 2026-03-27
tags: [标签 1, 标签 2]
---

# 文档标题

内容
\`;
  await app.vault.create('文件夹/文档名.md', content);
})()"

# 设置 frontmatter 属性
obsidian property:set name="tags" value="标签 1, 标签 2" file="文件夹/文档名.md"
```

## 输出文档模板

```markdown
---
title: [文档标题]
type: note
created: [日期]
tags: [相关标签]
---

# [文档标题]

> 来源：[文件名或网址]
> 整理日期：[日期]
> 类型：[图片/文档/网页]

---

## 核心信息

[一段话概括核心要点，海明威风格]

---

## 关键要点

1. **[要点 1]**
   [简洁描述]

2. **[要点 2]**
   [简洁描述]

---

## 总结

[简短总结]
```

## 海明威写作原则

**正确理解：**
- ✅ 保留全部信息，用更简洁的方式表达
- ❌ 不是挑选部分内容，不是删除信息

**技巧：**
- 短句：把长句拆成短句
- 主动语态：使用强动词
- 减少冗余：删除重复的词和无意义的修饰
- 一次一个想法：每个句子表达一个意思

**示例：**
- 原文："我在早上八点的时候起床，然后去吃了一个非常好吃的早餐，是本地的特色美食，是用大米做成的米线"
- 精简后："早上八点起床。去吃米线。本地特色美食。大米制成。"

## 文件清理

处理完成后，使用 `rm` 命令删除原文件：

```bash
# 删除处理过的文件
rm ~/Downloads/filename.ext
```

## 注意事项

1. **obsidian create 的局限**：只能创建到 vault 根目录，创建到子文件夹需用 `obsidian eval`
2. **网址处理**：必须先调用 `/markdown-proxy` skill 获取内容，不能直接访问网页
3. **文件删除**：确认内容已成功保存到 Obsidian 后再删除原文件
4. **重复检测**：如果文档已存在，添加时间戳或序号避免覆盖

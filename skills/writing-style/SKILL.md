---
name: writing-style
description: Writing style guide and format specifications for popular science articles. Auto-loaded during article writing, brainstorming, planning, reviewing, or translating content. Covers Chinese/English format specs (character limits for titles, subtitles, SEO descriptions, tags), writing conventions (plain language, analogies, avoiding jargon), and common pitfalls (ASCII box alignment for Chinese, nested code blocks).
user-invocable: false
---

# 科普文写作风格指南

这是给科普类公众号文章定的写作规范。涉及写作任务时（brainstorm、plan、写作、核查、翻译）请遵守以下要求。

## 总体风格

- **大白话科普**：面向对主题感兴趣但没有专业背景的普通读者
- **多用生活类比**：复杂概念先给类比，再讲细节
- **少用术语**：如果必须用，第一次出现时用一句话解释清楚
- **每篇 2000-3000 字**：过短不够深度，过长读者放弃
- **结构清晰**：用 `##` 分节，节内用列表或小段落

## 中文文章格式

### 标题
- 格式：`# 标题`
- 如果是系列文章：`# 系列名N: 标题`（例：`# Claude Code 进阶1: Context 管理`）

### 第一句（如果文章归属于某个合集）
- 系列文章格式：`本文是"[合集名]"合集 [系列名] 系列的第N篇。`
- 单篇文章格式：`本文是"[合集名]"合集的一篇。`（或类似自然表达）
- **合集名由用户指定**，已有合集示例："AI之旅"、"AI群星闪耀时"、"随笔之旅"
- 合集归属在 brainstorm 阶段决定，会记录到 brainstorm.md 和 plan.md 里
- 如果文章不属于任何合集，省略这句话

### 摘要
- 紧跟第一句之后（或标题之后，如果不是系列）
- 格式：`**摘要**：内容`
- 长度：100-120 字符（含标点和空格），**不超过 120**

### 日期声明（系列文章，可选）
- 紧跟摘要之后、正文第一节之前
- 格式：`> 本系列基于 YYYY 年 M 月 D 日的 XXX 官方文档。`

### 正文
- `##` 分节
- 不使用分割线 `---`
- 引出下一篇时自然过渡（如果是系列文章）

## 英文文章格式

### 标题
- 格式：`# Title`
- 长度：**小于 100 字符**
- 系列文章：`# Series Name N: Title`

### 第一句（如果文章归属于某个合集）
- 系列文章格式：`This is article N of the "[Collection]" collection, [Series Name] series.`
- 单篇文章格式：`This is a piece in the "[Collection]" collection.`（或类似自然表达）
- 合集名的英文译名由用户指定。示例映射：
  - "AI之旅" → "AI Journey"
  - "AI群星闪耀时" → 用户指定
  - "随笔之旅" → 用户指定
- 如果文章不属于任何合集，省略这句话

### Subtitle
- 格式：`**Subtitle:** 内容`
- 长度：**小于 140 字符**

### SEO Description
- 格式：`**SEO Description:** 内容`
- 长度：**小于 150 字符**

### SEO Tags
- 格式：`**SEO Tags:** Tag1 · Tag2 · Tag3 · Tag4 · Tag5`
- 数量：**5 个标签**，用 ` · ` 分隔

### Summary
- 格式：`**Summary:** 内容`
- 紧跟 SEO Tags 之后

### 正文
- `##` 分节
- 不使用分割线 `---`
- 内容与中文版对应，但**不是逐字翻译**，保持英文的自然表达
- 技术术语直接用英文原词

## 常见坑（必须避免）

### 1. ASCII 框图的中文对齐
中文字符宽度是英文的两倍，框图右侧竖线会对不齐。**中文内容的框图右侧不加 │**：

```
✗ 错误：
┌─────────────┐
│ 这是中文    │
└─────────────┘

✓ 正确：
┌─────────────
│ 这是中文
└─────────────
```

### 2. 代码块嵌套
Markdown 不支持代码块嵌套。如果需要在代码块里展示另一段代码，**把它们拆成独立的代码块**：

```
✗ 错误：
\`\`\`yaml
---
name: foo
---

运行脚本：
\`\`\`bash
./run.sh
\`\`\`
\`\`\`

✓ 正确：
\`\`\`yaml
---
name: foo
---
\`\`\`

运行脚本：
\`\`\`bash
./run.sh
\`\`\`
```

### 3. 不要乱用 emoji
除非用户明确要求，不要在文章里加 emoji。

### 4. 不要创建 README 或额外文档
除非明确要求，不要生成 README 或额外文档文件。

## 系列文章的额外约定

- 每篇结尾要**自然引出下一篇**
- 系列内文章不要重复核心内容，可以互相引用（"第 3 篇讲过……"）
- 系列之间允许有少量重叠，系列内不重复

## 写作流程

1. **先写中文 → 定稿** → 再写英文
2. **用户会说"定稿了"或"可以写英文"**，**不要自动触发英文版**
3. 有些文章用户可能不写英文版，用户会明确告知

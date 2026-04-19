---
name: check-format
description: Mechanically validate the format spec of a Chinese or English article — title length, subtitle/SEO description length, SEO tag count, series prefix, first sentence format, 摘要/Summary format, and other rules from the writing-style skill. Use after writing or translating an article to catch format violations before publishing. Reports pass/fail with specific violations.
argument-hint: [article file path]
---

# Check Format: 机械验证文章的格式规范

对一篇文章做**机械的格式合规检查**——字符数、标题格式、系列声明、摘要/Summary 格式、SEO 字段（英文）是否都对。产出通过/不通过报告。

参数 `$ARGUMENTS`：文章文件路径。**必须提供**，不猜。

## 检查对象

根据文件扩展名和内容判断是中文还是英文版。通常中文版没有"-en"后缀、有中文摘要；英文版有 Subtitle/SEO Description/SEO Tags 这些字段。

## 中文文章检查清单

| 检查项 | 规则 | 严重程度 |
|--------|------|---------|
| 标题格式 | 如果是系列文章：`# 系列名N: 标题`；单篇：`# 标题` | Critical |
| 系列声明 | 如属某合集：第一句 `本文是"[合集名]"合集 [系列名] 系列的第N篇。` | Critical（仅系列） |
| 摘要字符数 | 100-120 字符（含标点、空格） | Critical 若 >120；Warning 若 <100 |
| 摘要格式 | `**摘要**：内容`（**不**是 blockquote） | Critical 若格式错 |
| 日期声明位置（若有） | 在摘要之后、正文第一节之前 | Warning |
| 日期声明格式（若有） | `> 本系列基于 YYYY 年 M 月 D 日的 XXX 官方文档。`（blockquote） | Warning |
| 分节 | 用 `##`，不使用 `---` 分割线 | Critical 若有 `---` 作为分隔 |
| 结尾引导（仅系列非最终篇） | 自然引出下一篇 | Warning 若完全没有 |

## 英文文章检查清单

| 检查项 | 规则 | 严重程度 |
|--------|------|---------|
| Title 长度 | < 100 字符 | Critical |
| 标题系列前缀 | 若系列：`# Series Name N: Title` | Critical（仅系列） |
| 系列声明 | 如属某合集：第一句 `This is article N of the "[Collection]" collection, [Series Name] series.` | Critical（仅系列） |
| Subtitle | 格式 `**Subtitle:** 内容`；长度 < 140 字符 | Critical |
| SEO Description | 格式 `**SEO Description:** 内容`；长度 < 150 字符 | Critical |
| SEO Tags | 格式 `**SEO Tags:** Tag1 · Tag2 · ...`；恰好 5 个；用 ` · ` 分隔 | Critical 若数量不对，Warning 若分隔符错 |
| Summary 格式 | `> **Summary:** 内容`（blockquote） | Critical 若格式错 |
| 分节 | 用 `##`，不使用 `---` 分割线 | Critical 若有 |

## 工作流程

### 第 1 步：读文件

用 `Read` 工具读取 `$ARGUMENTS` 指定的文件。

### 第 2 步：判断语言和类型

- 看文件名（是否含 `-en` 或英文标题）
- 看前几行内容（中文字符占比）
- 判断是否系列文章：看标题是否有"进阶N:"、"第N篇"之类的前缀；或者第一句是否是系列声明模板

### 第 3 步：精确字符计数

Markdown 文本里的字符数必须用 Python 算准，**不要目测**。对关键字段用 Bash 执行：

```bash
python3 -c "s = '''<字段内容>'''; print(f'len: {len(s)}')"
```

提取字段时注意：
- 提取 `**摘要**：内容` 里的"内容"部分（去掉 `**摘要**：`）
- 提取 title 时去掉 `# ` 前缀
- 提取 Subtitle/SEO Description 时去掉 `**Subtitle:** ` 前缀
- 提取 Tags 后按 ` · ` 分割计数

### 第 4 步：逐项对照清单

每个检查项，给出 ✅ / 🟡 / 🔴 三档：

- ✅ 通过
- 🟡 Warning（非关键但该改）
- 🔴 Critical（必须修）

### 第 5 步：输出报告

```markdown
# Format Check Report

**文件**：<file_path>
**语言**：[中文 / 英文]
**类型**：[单篇 / 系列第 N 篇]
**检查日期**：YYYY-MM-DD

## 总结

- Critical 问题：N 条
- Warning 问题：N 条
- 通过：N 条

## 🔴 Critical 问题

### 1. [检查项名]

- **规则**：...
- **当前状态**：...（含测得的字符数等）
- **建议修改**：...

## 🟡 Warning 问题

### 1. ...

## ✅ 通过的检查

- [逐项列出通过的检查，简短]
```

## 与 review-cycle 的区别

- **check-format**：**机械**格式规则——字符数、前缀、标点。不关心内容对不对。
- **review-cycle 里的 fact-checker / academic-reviewer**：**内容**的事实和学术准确性。不关心标题多少字符。

两者互补——都应该在发布前跑过。

## 自动修复可选项

默认只报告不修复。如果用户明确说"顺便修复"，用 Edit 工具逐条修复 Critical，跳过 Warning（Warning 交用户自己判断）。修复后再跑一次 check-format 确认。

## 职责边界

- **不做内容核查**（交给 review-cycle）
- **不改写文章**（除非用户明确要求修复）
- **只读和报告**——检查本身是只读操作
- **不判断风格好坏**——只检查格式规则

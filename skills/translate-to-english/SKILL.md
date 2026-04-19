---
name: translate-to-english
description: Translate a finalized Chinese popular-science article to English. Not word-for-word — natural English expression preserving structure, analogies, and technical accuracy. Automatically handles format requirements (Title < 100, Subtitle < 140, SEO Description < 150, 5 Tags, Summary blockquote, series first sentence). Runs check-format at the end and retries fixes if format issues found. Only invoke after the user explicitly says the Chinese version is finalized and they want English.
argument-hint: [Chinese article file path]
---

# Translate to English: 把定稿的中文文章翻译成英文

把一篇**已定稿**的中文科普文章翻译成符合格式规范的英文版。保持结构、类比、技术准确性；英语自然表达而非逐字直译；自动生成 Title/Subtitle/SEO Description/SEO Tags/Summary 这些元数据。

参数 `$ARGUMENTS`：中文文章路径。**必须提供**。

## 前置条件（自查）

**只在用户明确要求时才运行**。不要在中文定稿后自动触发。用户会说"写英文"、"翻译成英文"、"可以写英文了"之类的。

读入文章前先确认：
- 用户说"定稿了"或"可以写英文"了吗？如果不确定，**先用 AskUserQuestion 问一次**。
- 中文文件确实存在且完整？

## 工作流程

### 第 1 步：读取上下文

1. **读中文文章**（`$ARGUMENTS`）——完整内容
2. **读同目录 plan.md**——拿以下信息：
   - 合集英文名（例："AI之旅" → "AI Journey"）
   - 如果系列：系列英文名
   - 本文在系列中的编号
   - 本篇的"关键概念"、"核查要点"（帮助翻译技术术语）
3. **读同目录其他已翻译的英文文件**（如果是系列且已有几篇）——保持术语一致

### 第 2 步：确定英文文件名

文件命名惯例：**kebab-case 英文 slug，放同目录**。例：

- `01-context管理.md` → `01-context-management.md`
- `12-Git-Worktree详解.md` → `12-git-worktree-deep-dive.md`

生成候选文件名（基于英文标题的 slug），**用 AskUserQuestion 让用户确认或修改**。不要直接写文件。

### 第 3 步：翻译原则

1. **结构保持一致**——章节名对应翻译，保持相同的 `##` 分节
2. **类比翻译**——类比是作者原创，要翻译出来，但用英语读者熟悉的措辞（例："一张书桌"→"a desk"，自然；避免生硬直译）
3. **技术术语保持英文原词**——LSP、MCP、Worktree、embedding、attention 等，不要翻译成中文含义的英文
4. **引用和链接**——URL 原样保留；引用的中文原文如果有必要，加上"(original in Chinese)"或直接删（看上下文）
5. **不做逐字直译**——中文常见的排比、"换句话说"、"简单说"这些过渡词，在英语里该换措辞就换
6. **长度可调整**——英语往往比中文略长（字数），但不要显著膨胀。原文 2500 字的文章，译文 ~2500 英文单词可以接受

### 第 4 步：生成英文元数据

按 `writing-style` 里英文格式规范：

- **Title**：< 100 字符。系列文章带前缀 `# Series Name N: Title`
- **系列声明**（如适用）：`This is article N of the "[Collection English]" collection, [Series Name English] series.`
- **Subtitle**：`**Subtitle:** 内容`，< 140 字符。浓缩本篇最大亮点
- **SEO Description**：`**SEO Description:** 内容`，< 150 字符。能让搜索用户一眼懂
- **SEO Tags**：`**SEO Tags:** Tag1 · Tag2 · Tag3 · Tag4 · Tag5`。恰好 5 个，用 ` · ` 分隔
- **Summary**：`> **Summary:** 内容` (blockquote)

### 第 5 步：写文件

用 `Write` 工具把翻译好的英文版写到第 2 步确认的路径。

### 第 6 步：自动运行 check-format

翻译完后立即调 `check-format` Skill 验证格式。**不要依赖用户来发现字符数超了。**

- 如果 check-format 全通过：告诉用户"英文版 `<path>` 生成完毕，格式合规"
- 如果有 Critical：
  - 根据具体问题修复（字符数超就精简、Tags 数量不对就补齐/删掉、格式错就改）
  - 再跑 check-format 验证
  - **最多重试 3 次**；3 次不通过就停下来问用户
- 如果只有 Warning：告诉用户通过但列出 warning 让他决定

### 第 7 步：更新 plan.md

英文版通过格式检查后，更新 plan.md 的进度追踪表：
- "英文状态" 列从 `未开始` 改为 `完成`（或"核查中"如果用户要跑 review-cycle）

### 第 8 步：告诉用户下一步

> 英文版已完成，保存在 `<path>`。格式检查通过。
>
> 如果你想对英文版也做事实/学术核查，跑 `/review-cycle <path>`。否则这篇就完成了。

**不要自动触发 review-cycle**——让用户决定。

## 翻译质量的几个具体约定

- **第一人称/第二人称**：中文喜欢"你"、"我们"；英语也自然用 "you" / "we"，可直接对应
- **反问句**：中文反问"这不是很明显吗？"，英语用 "Isn't it obvious?" 或干脆改陈述
- **成语/俗语**：中文成语找英语对应的表达，不要字面翻（例："一石二鸟"→ "kill two birds with one stone"；但"水土不服" 没直接对应，意译为 "doesn't fit here"）
- **技术缩写**：保持大写（LSP、MCP、API）
- **代码块**：代码原样保留，注释如果是中文，翻译成英文
- **表格**：表头翻译，内容翻译（除非是代码/命令/URL）

## 职责边界

- **只做翻译，不做内容创新**——不要在英文版里加中文版没有的论点
- **不做事实核查**（交给 review-cycle）
- **不自动触发 review-cycle**（让用户决定）
- **中文版不动**——翻译过程中发现中文版的问题，报告给用户，让用户决定是否改中文版；不要擅自改

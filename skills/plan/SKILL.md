---
name: plan
description: Generate a structured plan.md from an existing brainstorm.md for popular science writing. Use after /composir:brainstorm completes, or when the user has a brainstorm document and wants to produce an executable writing plan with article details, research needs, and progress tracking.
argument-hint: [optional path to brainstorm.md]
---

# Plan: 把 brainstorm 转成可执行的写作计划

读取 `brainstorm.md`，产出结构化的 `plan.md`——包含每篇文章的详细规划、关键术语、需要研究的内容、核查要点，以及进度追踪表。

参数 `$ARGUMENTS`：指向 brainstorm.md 的路径（形如 `<系列目录>/.composir/<slug>-brainstorm.md`）。如果省略，按以下顺序找：

1. glob 当前工作目录的 `.composir/*-brainstorm.md`
2. 如果恰好一个匹配，直接用
3. 如果多个匹配（同目录下多个单篇的 brainstorm），用 `AskUserQuestion` 让用户选
4. 如果 0 个匹配，问用户路径

## 流程

### 第 1 步：读取 brainstorm.md 并提取 slug

用 `Read` 工具读取 brainstorm.md 的完整内容。从**文件名**里提取 slug——文件名形如 `<slug>-brainstorm.md`，剥掉 `-brainstorm.md` 后缀即得 slug。这个 slug 会用在后面写出的 plan.md 文件名里。

确认它是由 `/composir:brainstorm` 生成的结构化文档。如果文件不存在或格式不匹配，**停下来问用户**："找不到 `.composir/<slug>-brainstorm.md`，请告诉我它在哪里，或者先运行 `/composir:brainstorm` 生成一份。"

### 第 2 步：检查信息完整性

快速扫描 brainstorm.md，确认以下信息都有了：

- [ ] 核心主题和切入角度
- [ ] 目标读者
- [ ] 形式（单篇 or 系列）
- [ ] 如果是系列：系列名、篇数、逻辑线、每篇初步主题
- [ ] 关键概念和需要核查的事实
- [ ] 如果涉及代码分析：代码库的本地路径（不能只有 GitHub URL）

**如果有缺失**，用 `AskUserQuestion` 补齐——不要自己脑补。

**特别地**，如果 brainstorm.md 显示主题涉及代码分析但只有 GitHub URL、没有本地路径：**停下来要求用户 clone 到本地**。理由同 brainstorm Skill（本地读文件比 WebFetch 抓 GitHub 页面高效得多）。拿到本地路径后再继续。

### 第 3 步：为每篇文章写详细规划

对每一篇文章，产出这几项：

1. **标题（中文）**：精炼、有吸引力的正式标题（不是 brainstorm 里的初稿）
2. **副标题**（如需要）
3. **核心问题**：这篇回答的那个"读者看完能懂什么"的具体问题
4. **关键概念**：需要讲清楚的 2-4 个概念
5. **主要类比**：这篇的"锚点类比"，贯穿全文
6. **结构大纲**：3-6 个 `##` 级章节标题
7. **关键术语/事实（需研究）**：哪些需要精确核查，写作时用 `/research` Skill 查权威来源
8. **核查要点**：fact-checker 和 academic-reviewer 需要特别关注什么
9. **结尾过渡**：如果是系列，怎么引出下一篇
10. **第一句模板**：基于合集归属（从 brainstorm.md 读取）生成。例：
    - 归属"AI之旅"合集的系列文章：`本文是"AI之旅"合集 系列名 系列的第N篇。`
    - 归属某合集的单篇：`本文是"[合集名]"合集的一篇。`
    - 不属于任何合集：省略这句

### 第 4 步：生成 plan.md

把所有内容整理成结构化 Markdown，保存到 brainstorm.md 所在的**同一个 `.composir/` 目录**，文件名为 `<slug>-plan.md`——其中 `<slug>` 和第 1 步从 brainstorm 文件名提取的 slug 一致。完整路径：`<系列目录>/.composir/<slug>-plan.md`。

### 第 5 步：提醒用户审核

输出完成后明确告诉用户："plan.md 已生成在 `<path>`，请审核——如果有需要调整的地方，直接告诉我修改哪里，或者用 `Ctrl+G` 之类的方式手动编辑。定稿后，我们可以开始写第一篇。"

**不要自动进入写作阶段**。

## plan.md 的结构模板

```markdown
# 写作计划：[主题/系列名]

**日期**：YYYY-MM-DD
**关联 brainstorm**：`./<slug>-brainstorm.md`（plan 和 brainstorm 同在 `.composir/` 下，共用 slug）
**状态**：plan 已生成，待用户审核

## 合集与系列

- **合集（中文）**：[合集名，如"AI之旅"；无合集则写"无"]
- **合集（英文）**：[英文译名，如"AI Journey"；无合集或不写英文则留空]
- **形式**：[单篇 / 系列（N 篇）]
- **系列名**：... （仅系列）
- **逻辑线**：... （仅系列）

## 定位

- **主题**：...
- **目标读者**：...
- **切入角度**：...

## 代码库位置（如果主题涉及代码分析）

- **GitHub URL**：...
- **本地路径**：...（写作阶段直接从这里读代码，不要用 WebFetch 去抓 GitHub）

如果主题不涉及代码分析，删除这一节。

## 写作约定

- 风格：大白话科普，多用类比，少用术语
- 字数：2000-3000 字/篇
- 中英文同步：先中文 → 用户定稿 → 再英文（用户明确要求后才写英文）
- 系列过渡：每篇结尾自然引出下一篇
- 格式规范：参见 `composir:writing-style` Skill

## 文章详细规划

### 第 1 篇：[标题]

- **副标题**（如需要）：...
- **核心问题**：...
- **关键概念**：
  - 概念 A：...
  - 概念 B：...
- **主要类比**：[主题] 像 [生活中的 Y]，因为 ...
- **结构大纲**：
  1. ## [第一节标题]
  2. ## [第二节标题]
  3. ## ...
- **关键术语/事实（需研究）**：
  - [术语 1]：需要查 [来源]
  - [事实 1]：需要核实 [具体内容]
- **核查要点**：
  - fact-checker 特别关注：...
  - academic-reviewer 特别关注：...
- **结尾过渡**：[怎么引出第 2 篇]（仅系列）

### 第 2 篇：[标题]

...（同上结构）

## 进度追踪

| # | 中文标题 | 中文状态 | 中文核查迭代 | 英文状态 | 备注 |
|---|---------|---------|-------------|---------|------|
| 1 | ... | 待写 | 0 / 5 | 未开始 | |
| 2 | ... | 待写 | 0 / 5 | 未开始 | |
| ... | ... | ... | ... | ... | |

### 状态值说明

- **中文状态**：待写 / 写作中 / 核查中 / 定稿
- **中文核查迭代**：`X / 5` 表示已迭代 X 次（最多 5 次，超过会问用户是否继续）
- **英文状态**：未开始 / 写作中 / 核查中 / 完成 / 不写（用户决定）

## 下一步

1. 用户审核本 plan.md，有需要调整的地方直接告知或手动编辑
2. 用户确认后，开始写第 1 篇中文版
3. 中文写完 → 用户追问/修改 → 定稿
4. 用户明确说"可以写英文"后再写英文版（或用户说不写英文则跳过）
5. 每完成一篇，更新本文档的"进度追踪"表
```

## 重要约定

- **plan 和 brainstorm 共用 slug**——从 brainstorm 文件名提取的 slug 直接用在 plan.md 上，不要另起名
- **plan.md 和 brainstorm.md 放同一个 `.composir/` 目录**——和文章本体分离，系列根目录只剩文章文件
- **每篇文章的"核查要点"要具体**，不要只写"检查事实"——越具体，后面 fact-checker 和 academic-reviewer 越能精准工作
- **"中文核查迭代 0 / 5"** 是批次 2 才用的字段，现在先写上占位，后面 review-cycle Skill 会实际去改这一列
- **不要自己决定每篇标题**——可以给候选，让用户拍板。如果用户没说清楚就暂时用"初稿标题"标记为 `[待定]`
- **写完 plan.md 就停**，不要开始写文章

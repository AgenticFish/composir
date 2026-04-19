---
name: research
description: Precise lookup for a term, concept, or fact using structured WebSearch + WebFetch. Use when writing an article and needing to verify or deepen understanding of a specific term (e.g., "what year did TypeScript 5.0 release?", "how does rust-analyzer compute borrow checker errors?"). Produces a concise Markdown summary with authoritative sources, ready to be cited or quoted. Prefer this over ad-hoc WebSearch when writing—it filters for authoritative sources.
argument-hint: [query or term]
---

# Research: 精确查找一个术语或事实

对一个具体术语/概念/事实做结构化查询，返回**简洁的结构化结果 + 权威来源 URL**。适合写作过程中遇到不确定的点、或主动核查一个关键事实。

参数 `$ARGUMENTS`：查询。例：
- `TypeScript 5.0 release date`
- `如何定义 embedding dimensionality`
- `rust-analyzer 用的是不是 salsa 增量编译`

## 工作原则

1. **权威源优先**——官方文档 > 一手论文/RFC > 权威教科书 > 公认技术博客 > 一般博客。引用时明确标注来源等级。
2. **时间敏感**——涉及版本号、API、工具能力时注意查最新。搜索词里加入年份或"latest"。
3. **对冲不同说法**——两个权威源冲突，如实报告冲突而不是自己选一个。
4. **不超出查询范围**——用户问了 A，不要顺便讲 A 的 10 个相关话题。如果有重要的 caveat 值得提醒，简短加一段。
5. **不要保存文件**——输出直接给 Claude 用于写作；除非用户明确要求保存到研究日志。

## 工作流程

### 第 1 步：理解查询

读用户的查询 `$ARGUMENTS`。如果模糊，**用 AskUserQuestion 澄清一次**——不要瞎猜猜偏。例：
- 用户问"attention 是什么"——太宽泛。追问："你想了解的 attention 是 Transformer 里的 self-attention、cross-attention、还是认知心理学里的 attention？是要一句话定义还是完整机制？"

### 第 2 步：结构化 WebSearch

不要直接丢原始查询进 WebSearch。**构造有针对性的搜索**：

- 涉及版本/产品：加"official"、"docs"，限定官方站点，例：`site:github.com OR site:microsoft.com typescript 5.0 release notes`
- 涉及概念：加"definition"、"paper"、"original"，找一手论文
- 涉及历史：加具体年份，加"announcement"、"release"
- 涉及 API：加"reference"、"documentation"

根据查询类型构造 1-3 次 WebSearch。

### 第 3 步：筛选和 WebFetch

从 WebSearch 结果里**手动挑出权威源**（看 URL 就能判断）：
- 官方站（github.com/org, docs.*, *.dev）
- 学术站（arxiv.org, acm.org, aclweb.org, proceedings.mlr.press）
- 知名机构（openai.com, anthropic.com, google-research.*, microsoft.com/research）

对 1-3 个最权威的结果用 WebFetch 读取完整内容。

### 第 4 步：交叉验证

如果找到的信息是关键事实（日期、数字、API 签名），**在第二个独立来源验证一次**。不一致就标注冲突。

### 第 5 步：产出结构化摘要

输出格式：

```markdown
# Research: [查询一句话]

## 一句话回答

[直接答案，≤ 50 字]

## 详细说明

[2-4 段，把关键细节和 context 讲清楚]

## 证据

- 🟢 [来源等级：官方 / 一手论文 / 权威教科书 / 知名博客]
  - URL: https://...
  - 关键引述："..."
  - 与本次查询的相关性：...

- 🟢 [另一个来源]
  - ...

## Caveats（如有）

- [重要的限定条件、版本差异、学界争议]

## 建议用法（在文章里怎么表述）

- [写成 "X 做 Y" 比较准确]
- [避免说 "Z"，因为 Z 其实不对]
```

**保持简洁**。写作时 Claude 会直接读这份摘要——太长反而不好用。

## 职责边界

- **不做事实核查**（那是 fact-checker 的事）——你是**写作前的 lookup 工具**，为写作提供准确信息
- **不评审文章**——你只回答查询，不看谁在用这个结果
- **只查 1-2 层深度**——查到答案就行，不要 rabbit hole 查 10 个相关问题
- **如果找不到权威源**——诚实报告"找不到一手源，以下是二手源说法，建议作者自己判断"，不要硬编答案

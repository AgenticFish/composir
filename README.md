# composir

Claude Code plugin for the popular-science writing workflow: brainstorm → plan → write → review.

## 安装

本插件通过 [AgenticFish marketplace](https://github.com/AgenticFish/marketplace) 分发。

```bash
# 在 Claude Code 里
/plugin marketplace add AgenticFish/marketplace
/plugin install composir@agenticfish
```

## 当前功能（三个批次完整）

### 批次 1：规划阶段

#### `/brainstorm`

驱动一次交互式头脑风暴，和你讨论：写什么、单篇 or 系列、系列结构、关键概念、类比等。还会和你一起列出本主题的**权威源白名单**（核查阶段用），并让你确认一个 slug（例：`git-worktree`、`ai-journey-android-skills`），产出 `.composir/<slug>-brainstorm.md`。如果主题涉及分析某代码库，会要求你把仓库 clone 到本地并记录 commit/tag。

#### `/plan`

读取 `.composir/<slug>-brainstorm.md`，产出结构化的 `.composir/<slug>-plan.md`——包含每篇文章的详细规划、关键术语、核查要点、权威源白名单、代码库位置（如适用）、进度追踪表。slug 和 brainstorm 一致。

#### 背景 Skill：`writing-style`

自动加载的风格指南。`user-invocable: false`——Claude 在涉及写作任务时自动加载，不需要你手动调用。

### 批次 2：核查循环

#### `/review-cycle [article-path]`

对写好的草稿执行完整核查循环：fact-checker 和 academic-reviewer 两个 agent **并行**审查，发现 Critical 自动修订，再次核查——最多 5 轮。

几个关键设计：

- **通过条件**：**Critical = 0 即通过**；Warning 降级为 advisory（合到报告里发给你参考，不触发下一轮），避免循环死磕"可以更精确"类 nuance
- **权威源白名单**：Critical 必须附一手源 URL（来自 plan.md 的"权威源"节或本地代码）；个人博客/Medium/Substack/SNS 最多支持 Warning
- **快照 + 增量核查**：每轮开始前存一份文章快照；iter2+ 只审本轮改动的段落 + 验证上轮 Critical 是否已修，不再重扫未动段落
- **禁止推翻上轮判定**：若 agent 认为上轮判错了，要在"对上轮判定的异议"节里说理由，由你裁决；不允许直接报相反 Critical
- **本地代码优先**：涉及目标仓库的断言必须用本地 Grep/Read；禁止对同仓库 WebFetch

每轮的报告和快照都保存在 `.composir/` 下，文件名带 `<article-slug>-review-*-iter<N>.md` 和 `<article-slug>-snapshot-iter<N>.md` 前缀。

#### Agent：`fact-checker`

独立事实核查员。验证具体事实——日期、名字、版本号、API 签名、引用——用 WebSearch 和 WebFetch 查一手源。只看事实，不管风格和学术框架。

#### Agent：`academic-reviewer`

学术/概念审查员。检查对概念和原理的解释是否符合学术共识，类比是否误导，有无过度简化到错的地步。不管事实细节。

### 批次 3：写作辅助工具

#### `/research [query]`

写作时的精确术语查询。结构化 WebSearch + WebFetch，过滤权威源，返回简洁摘要附上 URL。比直接 WebSearch 可靠，适合写作中遇到不确定的点。

#### `/check-format [article-path]`

机械的格式合规检查——字符数（摘要 100-120、英文 Title < 100、Subtitle < 140、SEO Description < 150）、Tags 数量、系列前缀、系列声明等。用 Python 精确计数，不目测。

#### `/translate-to-english [Chinese-article-path]`

中译英。不是逐字翻译——保持结构、类比、术语，用自然英语表达。自动生成 Title/Subtitle/SEO Description/Tags/Summary，完成后自动调 `check-format` 验证，不通过自动重试最多 3 次。**不自动触发**，仅在用户明确说"写英文"时运行。

## 工作流

```
/brainstorm                    ← 主题、读者、形式、结构、slug
   ↓ .composir/<slug>-brainstorm.md
/plan                          ← 结构化 plan
   ↓ .composir/<slug>-plan.md（用户审核）
[写作，中途可随时 /research <term> 查资料]
   ↓ 中文草稿（放在系列根目录）
/review-cycle article.md       ← 自动 fact + academic 核查 + 修订循环
   ↓ 通过后 .composir/<slug>-plan.md 标为"定稿"
/check-format article.md       ← 发布前机械格式检查
   ↓ 用户说"写英文"
/translate-to-english article.md
   ↓ 英文草稿（含自动 check-format）
/review-cycle english.md       ← 可选：英文也跑一遍核查
```

### 文件布局

所有 `.composir/` 下的文件都带**slug 前缀**，这样同一个目录下可以并存多个独立的 brainstorm/plan（适合把多篇单篇文章放一起）。

**系列的典型布局**（slug = 系列 slug）：

```
<系列目录>/
├── 01-xxx.md                                     ← 文章本体
├── 02-yyy.md
├── 01-xxx-english.md
└── .composir/
    ├── ai-journey-android-skills-brainstorm.md
    ├── ai-journey-android-skills-plan.md
    ├── 01-xxx-snapshot-iter1.md                  ← 每轮 agent 看到的版本
    ├── 01-xxx-snapshot-iter2.md
    ├── 01-xxx-review-fact-iter1.md
    ├── 01-xxx-review-academic-iter1.md
    ├── 01-xxx-review-fact-iter2.md
    └── 01-xxx-review-academic-iter2.md
```

**多个单篇共享目录**（每个单篇自己的 slug）：

```
<随笔目录>/
├── git-worktree.md                     ← 文章本体
├── ssh-config-tricks.md
└── .composir/
    ├── git-worktree-brainstorm.md
    ├── git-worktree-plan.md
    ├── git-worktree-review-fact-iter1.md
    ├── ssh-config-tricks-brainstorm.md
    └── ssh-config-tricks-plan.md
```

## License

MIT

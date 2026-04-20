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

驱动一次交互式头脑风暴，和你讨论：写什么、单篇 or 系列、系列结构、关键概念、类比等。产出 `.composir/brainstorm.md`。如果主题涉及分析某代码库，会要求你把仓库 clone 到本地。

#### `/plan`

读取 `.composir/brainstorm.md`，产出结构化的 `.composir/plan.md`——包含每篇文章的详细规划、关键术语、核查要点、进度追踪表。

#### 背景 Skill：`writing-style`

自动加载的风格指南。`user-invocable: false`——Claude 在涉及写作任务时自动加载，不需要你手动调用。

### 批次 2：核查循环

#### `/review-cycle [article-path]`

对写好的草稿执行完整核查循环：fact-checker 和 academic-reviewer 两个 agent 并行审查，自动应用修订，再次核查——最多 5 轮。超过 5 轮未通过会问你怎么办。每轮的报告都保存下来。

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
/brainstorm                    ← 主题、读者、形式、结构
   ↓ .composir/brainstorm.md
/plan                          ← 结构化 plan
   ↓ .composir/plan.md（用户审核）
[写作，中途可随时 /research <term> 查资料]
   ↓ 中文草稿（放在系列根目录）
/review-cycle article.md       ← 自动 fact + academic 核查 + 修订循环
   ↓ 通过后 .composir/plan.md 标为"定稿"
/check-format article.md       ← 发布前机械格式检查
   ↓ 用户说"写英文"
/translate-to-english article.md
   ↓ 英文草稿（含自动 check-format）
/review-cycle english.md       ← 可选：英文也跑一遍核查
```

### 文件布局

每个文章系列目录长这样：

```
<系列目录>/
├── 01-xxx.md                          ← 文章本体
├── 02-yyy.md
├── 01-xxx-english.md
└── .composir/                         ← 写作过程元信息（默认隐藏）
    ├── brainstorm.md
    ├── plan.md
    ├── review-fact-01-xxx-iter1.md
    └── review-academic-01-xxx-iter1.md
```

## License

MIT

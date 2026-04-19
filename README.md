# composir

Claude Code plugin for the popular-science writing workflow: brainstorm → plan → write → review.

## 安装

本插件通过 [AgenticFish marketplace](https://github.com/AgenticFish/marketplace) 分发。

```bash
# 在 Claude Code 里
/plugin marketplace add AgenticFish/marketplace
/plugin install composir@agenticfish
```

## 当前功能（批次 1 + 批次 2）

### 批次 1：规划阶段

#### `/brainstorm`

驱动一次交互式头脑风暴，和你讨论：写什么、单篇 or 系列、系列结构、关键概念、类比等。产出 `brainstorm.md`。如果主题涉及分析某代码库，会要求你把仓库 clone 到本地。

#### `/plan`

读取 brainstorm.md，产出结构化的 `plan.md`——包含每篇文章的详细规划、关键术语、核查要点、进度追踪表。

#### 背景 Skill：`writing-style`

自动加载的风格指南。`user-invocable: false`——Claude 在涉及写作任务时自动加载，不需要你手动调用。

### 批次 2：核查循环

#### `/review-cycle [article-path]`

对写好的草稿执行完整核查循环：fact-checker 和 academic-reviewer 两个 agent 并行审查，自动应用修订，再次核查——最多 5 轮。超过 5 轮未通过会问你怎么办。每轮的报告都保存下来（`review-fact-*.md`、`review-academic-*.md`）。

#### Agent：`fact-checker`

独立事实核查员。验证具体事实——日期、名字、版本号、API 签名、引用——用 WebSearch 和 WebFetch 查一手源。只看事实，不管风格和学术框架。

#### Agent：`academic-reviewer`

学术/概念审查员。检查对概念和原理的解释是否符合学术共识，类比是否误导，有无过度简化到错的地步。不管事实细节。

## 工作流

```
/brainstorm                   ← 主题、读者、形式、结构
   ↓ brainstorm.md
/plan                         ← 结构化 plan
   ↓ plan.md（用户审核）
[写作：批次 3 的 write-article Skill，或者手动写]
   ↓ 中文草稿
/review-cycle article.md      ← 自动 fact + academic 核查 + 修订循环
   ↓ 通过后 plan.md 标为"定稿"
用户说"写英文" → 英文版（批次 3 的 translate-to-english Skill）
```

## 计划中的功能（批次 3）

- `research`：精确术语/事实查询（结构化使用 WebSearch/WebFetch）
- `check-format`：字符数和格式规范验证
- `translate-to-english`：中译英（带格式验证）

## License

MIT

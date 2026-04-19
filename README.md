# composir

Claude Code plugin for the popular-science writing workflow: brainstorm → plan → write → review.

## 安装

本插件通过 [AgenticFish marketplace](https://github.com/AgenticFish/marketplace) 分发。

```bash
# 在 Claude Code 里
/plugin marketplace add AgenticFish/marketplace
/plugin install composir@agenticfish
```

## 当前功能（批次 1 MVP）

### `/composir:brainstorm`

驱动一次交互式头脑风暴，和你讨论：写什么、单篇 or 系列、系列结构、关键概念、类比等。产出 `brainstorm.md`。

### `/composir:plan`

读取 brainstorm.md，产出结构化的 `plan.md`——包含每篇文章的详细规划、关键术语、核查要点、进度追踪表。

### 背景 Skill：`writing-style`

自动加载的风格指南。`user-invocable: false`——Claude 在涉及写作任务时自动加载，不需要你手动调用。

## 工作流

```
/composir:brainstorm           ← 确定主题、读者、形式、结构
   ↓ 产出 brainstorm.md
/composir:plan                 ← 把 brainstorm 转成可执行的 plan
   ↓ 产出 plan.md（用户审核）
[写作：批次 3 将提供 write-article Skill]
   ↓ 中文草稿
[核查：批次 2 将提供 review-cycle]
   ↓ 定稿
用户说"写英文" → 英文版
```

## 计划中的功能

- **批次 2**：fact-checker、academic-reviewer 两个 agent + review-cycle Skill（自动事实/学术核查循环）
- **批次 3**：research Skill（精确术语/事实查询）、check-format Skill（字符数和格式验证）、translate-to-english Skill

## License

MIT

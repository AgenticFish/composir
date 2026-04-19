---
name: review-cycle
description: Orchestrate fact-check + academic-review + revision loop for a popular-science article. Spawns fact-checker and academic-reviewer agents in parallel to review a drafted article, collects their reports, applies fixes when issues are found, and re-reviews — up to 5 iterations. If still failing after 5, asks user whether to continue. Updates plan.md's progress table with iteration count.
argument-hint: [article file path]
---

# Review Cycle: 事实核查 + 学术核查 + 修订循环

对一篇已经写好草稿的科普文章执行完整的核查循环：fact-checker 和 academic-reviewer 两个子 agent 并行审查，发现问题自动修订，再次核查，最多迭代 5 次。

参数 `$ARGUMENTS`：文章文件路径。如果省略，自动从当前目录的 plan.md 里找"状态"是"核查中"或"写作中已完成"的那一篇。

## 流程

### 第 1 步：确定要审查的文章和同目录 plan.md

1. 如果用户给了 `$ARGUMENTS` 作为文件路径，直接用
2. 否则，在当前工作目录找 `plan.md`：
   - 如果找到，读它，看进度追踪表里哪一篇处于"写作中"或"核查中"状态
   - 如果多篇匹配，用 `AskUserQuestion` 让用户选
   - 如果 plan.md 不存在，**停下来问用户**："找不到 plan.md 也没给我文章路径。请告诉我要审查哪篇文章，或者先跑 `/plan` 生成计划。"

读取文章所在目录的 plan.md（用于后面更新进度和读"核查要点"）。

### 第 2 步：读当前迭代计数

从 plan.md 的进度追踪表里找这篇文章的"中文核查迭代"列，当前值记为 `N`（如 "2 / 5" 则 N=2）。

**如果 N >= 5**：
- 用 `AskUserQuestion` 询问用户：
  - "本篇已经核查了 5 次仍有问题。要继续迭代吗？"
  - 选 A：继续再迭代 5 次（N 归零重新计数）
  - 选 B：停下来，我手动处理
  - 选 C：接受当前版本，标为定稿（即使还有 warning）

根据选择决定是否继续。

### 第 3 步：并行触发两个核查 agent

**并行**触发 fact-checker 和 academic-reviewer——它们读同一篇文章但关注点不同，不互相依赖。用单次消息里的两个 Agent 工具调用同时发起。

给每个 agent 的 prompt 大致结构：

```
审查这篇文章：<绝对路径到文章>
参考同目录 plan.md 里"第 X 篇"的"核查要点"，特别关注那些点。

请按你的职责（fact-checker 的事实核查 / academic-reviewer 的概念学术核查）
产出结构化报告，遵守你的输出格式。
```

注意：给 agent 传绝对路径，避免它猜工作目录。

### 第 4 步：收集两份报告

两个 agent 完成后，它们会返回各自的 Markdown 报告。把两份报告**保存到文章同目录**：

- `review-fact-<article-name>-iter<N+1>.md`
- `review-academic-<article-name>-iter<N+1>.md`

保留历史报告，方便用户回看。

### 第 5 步：判断是否通过

扫描两份报告的"总结"部分：

- **通过**（不需要再修订）：**两份报告都没有 Critical，也没有 Warning**（Minor 不算）
- **不通过**：任意一份有 Critical 或 Warning

### 第 6 步：如果通过

1. 更新 plan.md：把这篇文章的"中文状态"从"核查中"改为"定稿"，迭代列填 `N+1 / 5`
2. 把两份报告的"Minor 问题"整合成一个 summary 发给用户参考
3. 告诉用户："本篇核查通过。要不要开始写英文版？（回复'写英文'开始，或'先不写'跳过）"——**不要自动触发英文**

### 第 7 步：如果不通过，自动修订

1. 读取两份报告的所有 Critical 和 Warning 条目
2. 对文章逐条应用修改：
   - 对每条 Critical/Warning，定位原文位置，用 Edit 工具改
   - 按报告里的"建议修改"执行；如果建议不清楚或有歧义，**保守修改**（倾向于加限定词、而非彻底改写）
3. 更新 plan.md 的迭代计数 `N → N+1`，状态保持"核查中"
4. 简短向用户报告："第 N+1 次迭代完成修订，包含：X 条 Critical、Y 条 Warning。重新进入核查……"
5. **回到第 2 步**（递归下一轮）

### 第 8 步：循环内的异常处理

- **报告解析不了**（agent 返回了非结构化内容）：停下来告诉用户，不要瞎修。
- **某个 Critical 的建议修改和原文冲突严重**（比如要删掉半篇文章）：停下来问用户，不要自动执行破坏性大的修订。
- **同一条问题多轮都没消**（agent 连续 2 轮报告同一个位置的同一个问题）：说明自动修订改不动，停下来问用户。

## plan.md 进度表的更新

找到进度表里这一行：

```
| 1 | 标题 | 核查中 | 2 / 5 | 未开始 | |
```

更新为：

```
| 1 | 标题 | 核查中 | 3 / 5 | 未开始 | |
```

或通过时：

```
| 1 | 标题 | 定稿 | 3 / 5 | 未开始 | |
```

用 Edit 工具精确修改这一行，不要重写整个表。

## 实用约定

- **两份报告始终保留**——即使通过了也不删，用户可能事后要看
- **自动修订要保守**——拿不准的改动宁可不改、让用户自己处理
- **每轮都更新 plan.md**——即使用户中途打断，下次继续也能知道到哪了
- **报告文件名带迭代号**——`review-fact-xxx-iter1.md`、`review-fact-xxx-iter2.md` ……方便对比看每轮的差异

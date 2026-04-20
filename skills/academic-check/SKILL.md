---
name: academic-check
description: Run academic-reviewer independently on a single article — audits concept explanations, analogies, and academic framing for correctness against research/literature consensus (does NOT verify raw facts — that's fact-checker's job). Auto-bootstraps .composir/<slug>-plan.md if missing by reading the article and proposing an authoritative-source whitelist for user confirmation. Use when you want to audit academic rigor without running the full review-cycle (no fact-checker in the loop). Iterates up to 5 rounds fixing Critical issues.
argument-hint: [article file path]
---

# Academic-Check: 单篇文章独立学术核查

对指定文章单独跑学术核查，不触发 fact-checker。适合：只想审概念框架和类比不要验具体事实、担心某个简化到"读者会被误导"的程度、或在还没完整 brainstorm/plan 的文章上做一次学术体检。

参数 `$ARGUMENTS`：文章文件路径（绝对或相对）。**必须传**——单 agent 模式不像 review-cycle 会从 plan.md 推断目标文章。

## 流程

### 第 1 步：定位文章 + 系列目录 + plan 路径

```bash
ARTICLE_PATH=$(realpath "$ARGUMENTS")
SERIES_DIR=$(dirname "$ARTICLE_PATH")
SLUG=$(basename "$ARTICLE_PATH" .md)
COMPOSIR_DIR="$SERIES_DIR/.composir"
PLAN_PATH="$COMPOSIR_DIR/${SLUG}-plan.md"
```

如果 `$ARGUMENTS` 为空或文件不存在，停下来问用户给路径，别瞎猜。

### 第 2 步：定位或 bootstrap plan.md

**分支 A — `$PLAN_PATH` 存在**：

读 plan.md，跳到第 3 步。

如果 plan.md 是 review-cycle/plan skill 生成的、**没有** `## 单 agent 核查历史` section，在 plan.md 末尾静默追加一个空的（用 Edit 追加，不打扰用户）：

```markdown

## 单 agent 核查历史

| 轮次 | 类型 | 日期 | Critical 数 | 状态 |
|------|------|------|-------------|------|
```

然后跳到第 3 步。

**分支 B — `$PLAN_PATH` 不存在（bootstrap）**：

1. Read 整篇文章，识别主题
2. 扫文章里的外部引用（产品名、项目名、论文、API、网址），列成候选
3. 基于主题推断**建议的权威源白名单**：
   - 讲某技术/产品 → 该技术/产品的官方网站、官方文档
   - 讲学术概念 → 原论文、权威综述、教科书（若有 URL）
   - 讲开源项目 → 官方 repo 的 README / docs / release notes（**但如果文章是分析该项目源码，按 CLAUDE.md §14 local-first 规则，还需要本地 clone 路径**）
4. 检测是否涉及代码分析：扫文章里的 github.com URL / "这个项目" / "源码" 等信号。若有，用 `AskUserQuestion` 问："文章涉及 <项目名> 源码分析，要给本地 clone 路径吗？（用于更精确的核查）"
5. 生成 plan.md 初稿（模板见下），用 Write 写入 `$PLAN_PATH`（先 `mkdir -p "$COMPOSIR_DIR"`）
6. 告诉用户：

   > 已在 `<PLAN_PATH>` 生成 plan 初稿。
   >
   > **请审阅"权威源"那一节**——academic-reviewer 只认白名单里的 URL / 域名作为 Critical 级别的证据。
   > 你可以增删域名、或跳过（空白名单时 agent 按通用黑名单兜底，但 Critical 判定会更保守）。
   >
   > 调整完告诉我"继续"。

7. **停下等用户说继续**（不要自动往下走）

**plan.md 初稿模板**：

````markdown
# <SLUG> plan

（本 plan.md 由 `/composir:academic-check` 自动生成。主要用途：让 fact-checker / academic-reviewer 拿到核查必需的"权威源"白名单。可随时手工编辑。）

## 权威源

（fact-checker / academic-reviewer 用这份白名单判 Critical——只有这些域名 / URL 出的信息能独立支撑 Critical 级别的判定。bootstrap 按主题推断了下列候选，请增删调整。）

- <推断域名 1>
- <推断域名 2>
- ...

## 核查要点

（可选，你想特别关注的断言或节。留空也 OK。）

## 代码库位置

（**仅当文章分析某开源项目代码时需要**。bootstrap 检测到下列 GitHub 引用：
- <检测到的 URL>

请补本地 clone 路径 + 基于的 commit/tag 用于版本锚定。如不涉及代码分析，删掉本节。）

- GitHub URL：<URL>
- 本地路径：<path>
- 基于 commit/tag：<tag 或 hash>

## 单 agent 核查历史

| 轮次 | 类型 | 日期 | Critical 数 | 状态 |
|------|------|------|-------------|------|
````

### 第 3 步：算本轮号 + 异常处理

读 `## 单 agent 核查历史` 表里 `type == academic` 的行：
- 已有 academic 行数 = `M`
- 本轮号 `N = M + 1`

如果 `M >= 1` 且最后一条 academic 行状态是 "pass"：用 `AskUserQuestion` 问 "上次已 pass，要强制再跑吗？"——否则退出。

如果 `N > 5`：用 `AskUserQuestion` 问 "本文已经跑了 5 轮 academic-check 还没收敛，要继续再跑 5 轮吗？"——选否则停，让用户手动处理。

**复用 review-cycle 第 9 步的异常处理**：
- 报告解析失败 → 停下问用户，不瞎修
- 单条 Critical 的建议修改影响过大（要删半篇）→ 停下问用户，不执行破坏性修订
- 同一条 Critical 连续 2 轮没消 → 停下问用户
- iter2+ 时 diff 为空（上轮后文章无改动）→ 停下问用户是否跳过本轮

### 第 4 步：本轮 snapshot + 准备 diff（iter2+）

1. **Snapshot**：用 Read 读 `$ARTICLE_PATH` 当前内容，用 Write 写到 `$COMPOSIR_DIR/${SLUG}-snapshot-academic-solo-iter${N}.md`
   - 文件名带 `-academic-solo-` 区分 review-cycle 的 `-snapshot-iter${N}.md`，互不覆盖

2. **Diff（仅 N >= 2）**：
   - 上轮 snapshot 路径：`$COMPOSIR_DIR/${SLUG}-snapshot-academic-solo-iter$((N-1)).md`
   - 上轮报告路径：`$COMPOSIR_DIR/${SLUG}-review-academic-solo-iter$((N-1)).md`
   - Bash 跑 `diff -u <上轮 snapshot> <当前文章>`，拿 unified diff 文本
   - 如果 diff 为空 → 异常处理（见第 3 步）

### 第 5 步：Dispatch academic-reviewer agent

**生成 prompt 前**：读 plan.md 的"代码库位置"节。若有本地路径，准备代码库片段追加到 prompt 末尾（同 review-cycle 第 4 步格式）：

```
本文涉及的代码库本地路径：<绝对路径>
（plan.md 记录的基于版本：<tag 或 commit hash>，如适用）

涉及该代码库的断言必须从这里读，不要用 WebFetch 抓 GitHub 或 WebSearch 找二手博客。
```

无本地路径则跳过这段。

**iter1 prompt**（N == 1）：

```
**CWD 必做**：先执行 `cd "<SERIES_DIR 绝对路径>"`，再跑任何 Bash（含 composir-fetch）。否则 `.composir/.cache/` 会写到错误目录，缓存永远命不中。

审查这篇文章：<ARTICLE_PATH 绝对路径>
参考 <PLAN_PATH 绝对路径> 里的"权威源"和"核查要点"节，特别关注那些点。

[如 plan.md 有代码库位置，追加代码库片段]

这是 academic-check **单 agent 模式**——只跑你一个，没有 fact-checker 并行；输出格式和规则同常规。

请按你的职责产出结构化学术核查报告。
```

**iter2+ prompt**（N >= 2）：

```
**CWD 必做**：先执行 `cd "<SERIES_DIR 绝对路径>"`，再跑任何 Bash（含 composir-fetch）。否则 `.composir/.cache/` 会写到错误目录，缓存永远命不中。

审查这篇文章：<ARTICLE_PATH 绝对路径>
参考 <PLAN_PATH 绝对路径> 里的"权威源"和"核查要点"节。

**本轮是第 <N> 轮 academic-check**（单 agent 模式——没有 fact-checker 的上轮报告）。

上一轮 academic-check 报告：<$COMPOSIR_DIR/${SLUG}-review-academic-solo-iter$((N-1)).md 绝对路径>
上一轮 academic-check snapshot：<$COMPOSIR_DIR/${SLUG}-snapshot-academic-solo-iter$((N-1)).md 绝对路径>

本轮文章与上轮 snapshot 的 diff（unified）：
```
<把第 4 步拿到的 diff 文本直接贴进来>
```

**核查指引**（iter2+）：

1. 重点审查本轮 diff 涉及的段落是否引入新的概念 / 类比 / 学术框架问题
2. 对上一轮报告里标出的 Critical，确认是否已修复（修复了就不用重报；没修或改错了才重报）
3. 未动过的段落默认延续上轮判定——除非本轮改动让其他段落的既有断言失真（跨引用才看），否则不要主动重新扫
4. 不要推翻上一轮的判断——若认为上轮某条 Critical 判错了，在报告里单独加一节 "## 对上轮判定的异议"
5. 同样遵守硬规则：Critical 必须附一手源 URL

[如 plan.md 有代码库位置，追加代码库片段]

请按你的职责产出结构化学术核查报告。
```

用 Task tool 的 `academic-reviewer` subagent 类型 dispatch。

### 第 6 步：保存报告 + 判断通过

Agent 返回 Markdown 报告。用 Write 存到 `$COMPOSIR_DIR/${SLUG}-review-academic-solo-iter${N}.md`。

扫报告的"总结"节：
- **Critical = 0** → 通过，进第 7 步
- **Critical > 0** → 进第 8 步（自动修订）

### 第 7 步：通过

1. 追加 `## 单 agent 核查历史` 表一行（用 Edit）：

   ```
   | N | academic | YYYY-MM-DD | 0 | pass |
   ```

2. 把报告里的 Warning + Minor 合成一段 advisory summary 发给用户，明说："这些是 advisory，不触发自动修订——你决定要不要处理。"

3. 告诉用户："Academic-check 通过，共 N 轮。报告路径列表：…"

### 第 8 步：不通过 → 自动修订

1. 读报告所有 Critical 条目
2. 逐条处理：
   - **没附一手源 URL 的 Critical** → **跳过**，累加到 `SKIPPED` 列表，在最终总结里汇报给用户（"agent 没遵守硬规则"）
   - 有一手源 → 按"建议修改"用 Edit 改文章；如果建议不清楚或有歧义，保守修改（加限定词、不彻底改写）
   - 如果某条 Critical 的建议改动太大（要删半篇）→ 停下问用户，别自动执行

3. 数：Critical 总数 `K`、修了 `FIXED = K - |SKIPPED|`、跳过 `|SKIPPED|` 条
4. 追加 `## 单 agent 核查历史` 表一行：

   ```
   | N | academic | YYYY-MM-DD | K | 修 FIXED，跳 |SKIPPED| |
   ```

5. 简短报告："第 N 轮 academic-check：K 条 Critical，修了 FIXED，跳过 |SKIPPED|（缺一手源）。进入第 N+1 轮……"
6. `N++`，回到第 3 步

### 第 9 步：循环结束后的最终总结

- 跑了几轮
- 最终状态（Critical=0 还是到 5 轮上限停）
- 所有报告路径列表（ iter1 → iterN）
- 未修的 Critical 汇总（若有 SKIPPED）
- 最新的 advisory（Warning + Minor）

## 与 review-cycle / fact-check 的关系

- `/composir:review-cycle` —— fact + academic 并行，full flow；写 plan.md 的"进度追踪表"（列 `中文核查迭代`）
- `/composir:fact-check` —— 只 fact 单边；写 plan.md 的 `## 单 agent 核查历史` 表 type=fact 行
- `/composir:academic-check` —— 只 academic 单边；写同一张表 type=academic 行
- 三者使用的 snapshot / report 文件名互不冲突：
  - review-cycle：`${SLUG}-snapshot-iter${N}.md` / `${SLUG}-review-fact-iter${N}.md` / `${SLUG}-review-academic-iter${N}.md`
  - fact-check skill：`${SLUG}-snapshot-fact-solo-iter${N}.md` / `${SLUG}-review-fact-solo-iter${N}.md`
  - academic-check skill：`${SLUG}-snapshot-academic-solo-iter${N}.md` / `${SLUG}-review-academic-solo-iter${N}.md`
  - 三条迭代序列独立计数，不共享 N

## 实用约定

- 报告和 snapshot 一律保留，不删
- 自动修订要保守——拿不准宁可不改、让用户处理
- 每轮都更新 plan.md，即使用户中途打断也能续
- 用 Edit（不要 Write）追加/修改 plan.md 的表，避免覆盖其他内容

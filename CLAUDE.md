# composir — Claude Code Plugin for Popular-Science Writing Workflow

## What this is

Claude Code plugin that drives the full writing workflow for popular-science articles: **brainstorm → plan → write (with research) → review-cycle → format check → translate-to-english**.

- **Plugin repo**: https://github.com/AgenticFish/composir (this repo)
- **Marketplace repo**: https://github.com/AgenticFish/marketplace
- **Marketplace name** (for install commands): `agenticfish`
- **Install**: `/plugin marketplace add AgenticFish/marketplace` + `/plugin install composir@agenticfish`

## Architecture

Three MVP batches + one polish/hardening round, each a minor version bump:

| Batch | Version | Components |
|-------|---------|-----------|
| 1 — Planning | 0.1.0 → 0.1.1 | `brainstorm`, `plan`, `writing-style` (background, auto-load) |
| 2 — Review | 0.2.0 | `review-cycle` orchestrator + `fact-checker` & `academic-reviewer` agents |
| 3 — Writing tools | 0.3.0 | `research`, `check-format`, `translate-to-english` |
| 4 — Polish & hardening | 0.4.0 | `.composir/` folder + slug-prefixed filenames + review-cycle hardening (no new skills/agents; behavior & structure changes only) |
| 5 — Webfetch cache | 0.5.0 | `bin/composir-fetch` + 5 处 skill/agent 接入三层缓存（behavior change, no new skills/agents） |

File layout:
- `.claude-plugin/plugin.json` — plugin manifest (name, version)
- `skills/<name>/SKILL.md` — each skill
- `agents/<name>.md` — each subagent definition
- `README.md` — user-facing documentation

## Key design decisions

Finalized 2026-04-18:

1. **Process-artifact location**: each series gets a `.composir/` subdirectory holding brainstorm, plan, and review reports. Articles themselves stay in the series root directory. Rationale: keeps articles visually separate from process metadata (2026-04-19).
2. **Slug-prefixed filenames inside `.composir/`** (2026-04-19): all files use a slug prefix, so one `.composir/` can hold multiple independent writing units without collision. Naming:
   - `<slug>-brainstorm.md`, `<slug>-plan.md` — slug = series slug (series mode) or article slug (single-article mode, multiple singles sharing one dir)
   - `<article-slug>-review-fact-iter<N>.md`, `<article-slug>-review-academic-iter<N>.md` — always per-article
   - Lookup: plan skill globs `*-brainstorm.md`; review-cycle/translate try `<article-slug>-plan.md` first then glob `*-plan.md`
3. **brainstorm.md**: persisted and committed (not ephemeral)
4. **Iteration counter**: stored in plan.md's progress table (not a separate state file)
5. **Fact vs academic check**: two independent agents with separate forked contexts
6. **Post-review flow**: stops after Chinese finalized — English is only written when user explicitly asks; **do not auto-trigger English**
7. **Legacy-series fallback**: skills read ONLY from the current `.composir/` layout with slug prefixes. Pre-`.composir/` series (flat layout) and pre-slug-prefix series (un-prefixed `brainstorm.md`/`plan.md` inside `.composir/`) need manual migration if the user wants to keep running skills on them; otherwise they're left as-is.

### Review-cycle hardening (2026-04-19, shipped in 0.4.0)

Driven by a real-world case where `aizhilv/claude-code/01-context管理` hit 4 iterations without converging — agents cited personal blogs as "authoritative", contradicted each other on numbers, and the pass condition required both Critical=0 and Warning=0, which was nearly impossible when each round invented new "could be more precise" nuances.

8. **Per-topic authoritative source whitelist, not plugin-level**: popular-science topics are too varied for a static plugin-wide list; brainstorm asks the user to confirm a per-topic whitelist that gets carried through plan.md. Agents read plan.md's `## 权威源` section. A type-level blacklist (personal blogs, SNS, forum posts) lives in agent definitions as general principle — not a domain list.
9. **Critical=0 is the pass condition; Warning is advisory**: Warning/Minor no longer block passing. They get surfaced as a summary for the author to consider. This directly breaks the infinite-nuance loop.
10. **Critical requires a first-hand source URL**: enforced in both agent definitions. Without one, a Critical claim must be downgraded. review-cycle's auto-revision also skips Criticals that lack first-hand sources and flags the omission to the user.
11. **Snapshots + incremental review for iter2+**: before invoking agents, review-cycle saves `.composir/<article-slug>-snapshot-iter<N+1>.md`. iter2+ agents receive a unified diff (vs prior snapshot) + paths to prior reports, and are instructed to only audit touched paragraphs + verify prior Criticals were fixed + do cross-reference checks when a diff change obviously affects an un-touched paragraph. No re-scanning for fresh nuances in untouched text.
12. **No direct contradictions of prior-round judgments**: if an agent disagrees with a prior Critical, it writes a `## 对上轮判定的异议` section rather than reporting an opposite Critical. review-cycle/author arbitrates. Prevents ping-pong between rounds.
13. **Stricter over-simplification bar (academic-reviewer)**: "reader's mental model leads to a wrong conclusion" is the Critical line. Simplifications that are directionally correct — even if imprecise — are at most Minor. Self-check: "in what concrete decision would the reader act wrong under the current explanation?" — if no concrete wrong action, not a Critical/Warning.
14. **Local-repo-first for code analysis**: when plan.md records a local clone path, agents MUST use Grep/Glob/Read locally for code/API/config claims. WebFetch against the same GitHub repo and WebSearch for second-hand blogs about it are disallowed (PR comments, issue threads, release notes are exceptions since they're not in the code). brainstorm also captures commit/tag to let agents verify on the intended version.

### Webfetch 缓存 (2026-04-20, 0.5.0)

驱动：review-cycle 多轮迭代时 fact-checker / academic-reviewer 对同一批 URL 反复 WebFetch；brainstorm 抓过的页不被下游 agent 复用——浪费 token、拖慢核查。

15. **三层缓存 + 脚本契约**：`.composir/.cache/` 下建持久缓存；插件 ship `bin/composir-fetch <url>` 脚本，skills/agents 通过 Bash 调脚本而非直接 WebFetch。三层命中：(a) raw-page HTML（curl 存，只缓存 2xx）；(b) WebFetch Q/A 日志（`<hash>.wf.md` 按 URL 累积历次回答）；(c) 都 miss → WebFetch 并追加 Q/A。永不过期（系列目录天然隔离）；`.gitignore` 自举排除。
16. **canonical block 在 5 个触点重复、不抽象**：Claude Code skill/agent 文件是独立上下文信封，没有 include 机制。fetch 协议只能在 5 个文件里各贴一份；验证通过比对 block 的 SHA 保证同步。
17. **例外：本地代码优先**：plan.md 登记的本地仓库对应的 GitHub URL 仍按 local-first 规则（§14）走本地 Grep/Read，不经缓存。

## Plugin vs memory split

- **Plugin (this repo)**: the writing system's engine — skills, agents, format rules, workflow logic. Shared, version-controlled.
- **Memory** (`~/.claude/projects/.../memory/`): user-level preferences and dynamic state (per-series progress that fell back to memory, writing feedback). Personal, not shareable.

Rule of thumb: if it belongs in "how the system works," it's in the plugin. If it's "what is currently happening," it's in memory or in the series' own `.composir/<slug>-plan.md`.

## Development conventions

### Commit format
Conventional-style: short summary line + bullet details. End with:
```
Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
```

### PR flow
- Branch → PR → merge on GitHub
- Each content change = separate PR; version bump = separate PR on top

### Version bumps (REQUIRED)
Claude Code uses `plugin.json`'s `version` for update detection. **Without a bump, `/plugin marketplace update` skips the change.**

- **patch** (0.x.Y): small fixes
- **minor** (0.Y.0): new features, backward compatible — used for each batch
- **major** (X.0.0): breaking changes

### After a merge
In any Claude Code session:
```
/plugin marketplace update agenticfish
/reload-plugins
```

## Version history

- **0.1.0** — Batch 1: brainstorm, plan, writing-style
- **0.1.1** — Code-analysis topics require local clone before brainstorming (prevents WebFetch-against-GitHub inefficiency)
- **0.2.0** — Batch 2: review-cycle with fact-checker + academic-reviewer agents
- **0.3.0** — Batch 3: research, check-format, translate-to-english
- **0.4.0** — `.composir/` folder for process artifacts + slug-prefixed filenames + review-cycle hardening (per-topic source whitelist, Critical=0 pass condition, snapshots + incremental review, prior-round context, stricter over-simplification bar, local-repo-first for code analysis)
- **0.4.1** — brainstorm actively asks "run /composir:plan now?" and invokes the plan skill on confirmation, instead of ending with a passive "随时 /composir:plan" message that forces the user to type the command by hand
- **0.5.0** — 三层 WebFetch 缓存：`bin/composir-fetch` 脚本（只缓存 2xx 响应） + 5 处 skill/agent 接入；raw page + WebFetch Q/A 累积日志；`.composir/.cache/` 系列-local，gitignore 排除

## Collections referenced in the plugin

Collections the user publishes popular-science articles to:
- "AI之旅" / "AI Journey" (primary so far)
- "AI群星闪耀时" (English TBD)
- "随笔之旅" (English TBD)

`brainstorm` always asks the user for the collection — not hardcoded.

## Related project

User's main writing repo: `/Users/irene.yu/Documents/workspaces/IreneXY/misc/weichat/aizhilv/` — where articles written with this plugin live. Writing style specs in the `writing-style` skill were derived from that repo's accumulated conventions.

## When to consult this CLAUDE.md vs skip

**Consult when**: user is modifying the plugin — adding skills/agents, changing workflow, bumping versions, writing README, discussing architecture.

**Skip when**: user is using the plugin (running `/brainstorm`, `/plan`, etc. in a different project) — the skills handle their own instructions.

## Open questions / decided non-goals

- **Migration of existing memory series data into per-series plan.md files**: NOT planned (user decided 2026-04-18)
- **Auto-migration of pre-`.composir/` flat-layout series**: NOT planned (user decided 2026-04-19). Those series either stay flat or are manually moved by the user.
- **Auto-migration of un-prefixed `.composir/brainstorm.md`/`plan.md` from the short-lived intermediate layout**: NOT planned. Rename to `<slug>-brainstorm.md`/`<slug>-plan.md` manually if the skills need to read them.
- **Hook-based automation** (e.g., PostToolUse auto-run check-format): NOT currently planned
- **End-to-end walkthrough documentation**: NOT currently planned; README covers basics

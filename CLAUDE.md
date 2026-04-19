# composir — Claude Code Plugin for Popular-Science Writing Workflow

## What this is

Claude Code plugin that drives the full writing workflow for popular-science articles: **brainstorm → plan → write (with research) → review-cycle → format check → translate-to-english**.

- **Plugin repo**: https://github.com/AgenticFish/composir (this repo)
- **Marketplace repo**: https://github.com/AgenticFish/marketplace
- **Marketplace name** (for install commands): `agenticfish`
- **Install**: `/plugin marketplace add AgenticFish/marketplace` + `/plugin install composir@agenticfish`

## Architecture

Three batches, each a minor version bump:

| Batch | Version | Components |
|-------|---------|-----------|
| 1 — Planning | 0.1.0 → 0.1.1 | `brainstorm`, `plan`, `writing-style` (background, auto-load) |
| 2 — Review | 0.2.0 | `review-cycle` orchestrator + `fact-checker` & `academic-reviewer` agents |
| 3 — Writing tools | 0.3.0 | `research`, `check-format`, `translate-to-english` |

File layout:
- `.claude-plugin/plugin.json` — plugin manifest (name, version)
- `skills/<name>/SKILL.md` — each skill
- `agents/<name>.md` — each subagent definition
- `README.md` — user-facing documentation

## Key design decisions

Finalized 2026-04-18:

1. **plan.md location**: each series gets its own `plan.md` in the series directory, co-located with articles (flat)
2. **brainstorm.md**: persisted and committed (not ephemeral)
3. **Iteration counter**: stored in plan.md's progress table (not a separate state file)
4. **Fact vs academic check**: two independent agents with separate forked contexts
5. **Post-review flow**: stops after Chinese finalized — English is only written when user explicitly asks; **do not auto-trigger English**

## Plugin vs memory split

- **Plugin (this repo)**: the writing system's engine — skills, agents, format rules, workflow logic. Shared, version-controlled.
- **Memory** (`~/.claude/projects/.../memory/`): user-level preferences and dynamic state (per-series progress that fell back to memory, writing feedback). Personal, not shareable.

Rule of thumb: if it belongs in "how the system works," it's in the plugin. If it's "what is currently happening," it's in memory or in the series' own plan.md.

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
- **Hook-based automation** (e.g., PostToolUse auto-run check-format): NOT currently planned
- **End-to-end walkthrough documentation**: NOT currently planned; README covers basics

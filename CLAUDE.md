@AGENTS.md

## Claude Code notes

- Skills live in `.claude/skills/` (`ui-presentation-audit`, `ui-ux-pro`). Subagents live in `.claude/agents/`.
- Delivery follows the one-pass flow in `AGENTS.md`: start from the current base branch, implement the whole scope,
  build once, run the minimum affected tests once, fix and re-run in the same pass, audit user-facing changes before
  committing, then report once with the failure log and a one-line `Performance:` result. Do not use `AskUserQuestion`
  for non-material choices; an explicit instruction in the request overrides these defaults.
- Commit task-related files only and push to the branch the task designates. Open a pull request, merge, deploy or write
  to an external system only when the request asks for it.
- Ship capabilities on, and guard what is not ready (`AGENTS.md`): a feature that depends on a service that may not be
  ready ships switched on behind one capability check (Available, Needs setup, Unavailable, Failing) that falls back to
  the manual path without losing the user's work and says so in plain words. It never turns on automatic effects that
  spend or confirm money.
- Default performance check (`AGENTS.md`): for every work unit, triage the change for a measurable read or write path,
  request, query, payload or rendered list; measure before and after only when one is touched; never claim an
  improvement without numbers. Brief subagents with it.
- Money is minor units (`long`) plus a currency code, quantities are `decimal`, financial records are voided and never
  deleted, and every query is tenant-scoped (`AGENTS.md` sections 6, 7 and 9). Treat a change to any of these as
  Complex-tier work.
- Run no background or wide test commands, and tell subagents the same.

## Model routing

Cost is a hard constraint. Use the cheapest current Claude model that is good at the job. The session
default for agentic IDE work in this repo is Sonnet 5.5, including when an account or vendor default
would start on Opus. Do not route everyday coding to Opus or Fable.

**Sonnet does the daily work. Haiku does the narrow work. Opus steps up for the hard calls. Fable is a
last resort, never the default.**

November 2026 lineup. Each model has a 1M context window and 128K max output. `.claude/settings.json`
pins the aliases to these Anthropic API IDs. `haiku` is also the background model (compaction and
other small fast work). A provider that rejects these IDs can pin its own in the uncommitted
`.claude/settings.local.json`. The Agent tool's `model` argument accepts only the aliases `haiku`,
`sonnet`, `opus`, and `fable`.

| Class | ID | Aliases | USD per MTok, input / output | Use for |
| --- | --- | --- | --- | --- |
| Sonnet 5.5 | `claude-sonnet-5-5` | `sonnet` | $2 / $10 | Default session model. Well-scoped features, bug fixes, UI polish, everyday multi-file edits, fast iteration. |
| Haiku 5.5 | `claude-haiku-5-5` | `haiku` | $0.10 / $0.50 up to 100K prompt tokens; $0.50 / $2.50 above that | Subagents with no stronger pin, compaction, summaries, classification, lookups, narrow browser replays. |
| Opus 5.5 | `claude-opus-5-5` | `opus`, `best` | $4 / $20 | Complex, open-ended, or codebase-wide work that needs sustained judgment. |
| Fable 5.2 | `claude-fable-5-2` | `fable` | $8 / $40 | Rare, hard, ambiguous, multi-hour work. Never the session default. |

Each specialist sets `model` and `effort` in its frontmatter. That pin wins over the parent session, so
a subagent does not inherit Opus or Fable unless its row below says it should. Subagents with no model
of their own use Haiku 5.5 (`CLAUDE_CODE_SUBAGENT_MODEL`). The read-only `Explore` agent is pinned to
Haiku for file and symbol lookups. The built-in Plan subagent follows the session model, so plan-mode
research stays on Opus when the session is `opusplan`.

When a session needs a stronger plan than Sonnet, switch that session to `opusplan` (Opus while
planning, Sonnet while implementing). Do that instead of leaving the whole session on Opus.

Raise effort from `medium` to `high` or `xhigh` before stepping up a model when the miss looks like
skipped files, tests that were not run, or a refactor abandoned midway. Step up a model only after the
agent had the context, clearly tried, and was still wrong.

Step up to Opus 5.5 for architecture that needs sustained judgment, a large refactor, a difficult
review, or a long session where Sonnet at higher effort was still wrong. Reserve Fable 5.2 for a
migration that would otherwise stall, root-cause or outage debugging, architecting work that would
normally be split into pieces, or orchestrating smaller models. Do not use Fable for a routine change
or a small feature. Pass `model: "fable"` only after an Opus run at `xhigh` was still wrong, and say so
in the report. Keep Sonnet implementers off `max` effort, where they sometimes make out-of-scope changes.

Older 4.x, 5, and 5.1 IDs (Opus 4.x, Sonnet 4.5, Haiku 4.5, Fable 5, Fable 5.1, and the rest) are
legacy. Do not select them for work in this repo.

| Agent | Model, effort | Why |
| --- | --- | --- |
| `solution-architect` | Opus 5.5 (`claude-opus-5-5`), `xhigh` | Architecture and impact plans. A wrong plan costs the most |
| `dev-lead-reviewer`, `security-engineer` | Opus 5.5 (`claude-opus-5-5`), `xhigh` | Difficult review of complex or security-sensitive changes |
| `product-planner` | Opus 5.5 (`claude-opus-5-5`), `high` | Open-ended requirements and backlog decomposition |
| `integration-engineer` | Opus 5.5 (`claude-opus-5-5`), `high` | Parallel writers, migrations, and shared surfaces |
| `ui-ux-expert`, `ux-auditor` | Opus 5.5 (`claude-opus-5-5`), `high` | Difficult visual or product-wide UX judgment. Minor UI polish stays on the Sonnet session |
| `senior-engineer` | Sonnet 5.5 (`claude-sonnet-5-5`), `high` | Implementation from a settled plan: APIs, Angular UI, tests, routine fixes |
| `quality-engineer` | Sonnet 5.5 (`claude-sonnet-5-5`), `high` | Acceptance-test matrix and failure judgment |
| `qa-verifier` | Haiku 5.5 (`claude-haiku-5-5`), `medium` | Replay of a documented journey. Narrow browser work |
| `Explore`, `mechanical-extractor` | Haiku 5.5 (`claude-haiku-5-5`), `medium` | Lookups, transcription, inventories, summaries. Read-only |

| Trigger | Agent | Skip when |
| --- | --- | --- |
| Outcome, scope, business rules or acceptance criteria are unresolved, or broad work needs decomposing | `product-planner` | Requirements are already clear and testable |
| Before any edit on Complex work: uncertain root cause, cross-component change, migration, concurrency, tenant or public-contract change | `solution-architect` | The implementation path is already clear |
| A settled plan to implement | `senior-engineer` on Sonnet 5.5 | The main session can do Small or Normal work directly. Pass `model: "opus"` only after Sonnet at `high` had the context, tried, and was still wrong |
| Parallel writers explicitly authorized, with an ownership ledger | `integration-engineer` | Always, unless parallel writing was authorized |
| A new or materially changed user-facing workflow, or unclear interaction requirements | `ui-ux-expert` | Minor copy, styling, or a familiar component |
| Complex or multi-layer work that has passed validation, before reporting done | `dev-lead-reviewer` | Small, low-risk edits |
| A change touching authentication, authorization, tenant isolation, PII, payroll, uploads, the public API, secrets or dependencies | `security-engineer` | No security or privacy boundary changed |
| Independent acceptance testing wanted after implementation | `quality-engineer` | Focused tests already give enough confidence |
| Replaying a documented journey after a fix | `qa-verifier` | It is the original exploration |
| File or symbol lookup, or mechanical transcription with no judgment | `Explore` or `mechanical-extractor` | The task needs architecture, visual judgment, or a review |

Run the reviewers (`dev-lead-reviewer`, `security-engineer`) in parallel after validation passes. For a
one-off subagent, pass `model: "haiku"` for a read-only lookup, summary, classification, or replay;
pass `model: "sonnet"` when it must implement; pass `model: "opus"` only for the Opus cases above.

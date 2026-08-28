# poteto-mode: pstack port to Claude Code — Feature Plan

Port Lauren Tan's pstack (cursor/plugins `pstack/`, v0.14.5, MIT) to a Claude Code plugin.
Fresh port from upstream; the community ports (pstack-claude, open-pstack) are reference
material for mapping decisions, not a base.

## Locked decisions

| Decision | Choice |
|---|---|
| Build base | Fresh from upstream v0.14.5 |
| Models | Claude-only: Sonnet 5, Opus 5, Fable 5 (no Haiku), highest effort where the role demands it |
| Distribution | This repo is a plugin marketplace; installed globally, works in every project |
| Sticky mode | SessionStart hook injects the routing mandate (startup, clear, compact) |
| Autonomous ceiling | Merge-ready, never merge. Interrogate + blast-radius + babysit (CI, review threads) precede it |
| Stacking | No Graphite. Plain `gh`; stacked-PR mechanics reimplemented on gh base/head graph |
| PR bots | CodeRabbit (auto) + Claude review triggered by babysit posting `@claude review` |
| Automations | benny dropped (no Slack) |
| Runtime | bun on Windows 11; scripts stay TypeScript/bun |

## Model role map (replaces Cursor's grok/sol/fable/opus)

| pstack role | Cursor default | Ours |
|---|---|---|
| Fast mechanical code (feature, refactoring delegates; swarm workers; live lanes; how explorers; why investigators) | grok-4.6-fast-xhigh | **sonnet-5** (high effort) |
| Precise instruction-following code (bug-fix, perf, hillclimb delegates) | gpt-5.6-sol-max | **opus-5** (max effort) |
| Judgment, prose, hardest changes; synthesizers; leads | claude-fable-5-thinking-max | **fable-5** (max effort) |
| Review panels (interrogate reviewers, how critics, arena runners, architect runners) | 4-vendor panel | **fable-5 + opus-5 + sonnet-5** (3 lanes) |
| "Different model family" judge/verifier rule | cross-vendor | **different tier than the worker** (e.g. sonnet worker → opus/fable judge) |

Effort pinning: effort-pinned agent definitions (`pstack-fable-max`, `pstack-opus-high`, …)
per open-pstack's pattern, since the Agent tool takes a model but effort lives in the agent def.
Config file: `~/.claude/pstack-models.md`, written by `/setup-pstack`, imported from user
CLAUDE.md via `@` import; every skill reads roles from it and falls back to the defaults above.

## Repo layout (target)

```
poteto-mode/
  .claude-plugin/marketplace.json
  plugins/pstack/
    .claude-plugin/plugin.json          # name, version tracking upstream 0.14.5, MIT + attribution
    hooks/                               # SessionStart sticky-mode hook + cross-platform runner
    agents/                              # poteto-agent, comment-sicko, effort-pinned lanes
    skills/
      poteto-mode/                       # router SKILL.md + playbooks/ + references/ + scripts/
      <22 workflow skills>/
      principle-*/                       # 21 principles, user-invocable: false
  PLAN.md  LICENSE  NOTICE.md  README.md
```

Lessons adopted from the community ports (their changelogs document the failures):
no command trampolines next to same-named skills; `user-invocable: false` on principle
leaves; never `disable-model-invocation` on skills the router must invoke; no
cross-marketplace `dependencies` in the manifest; `${CLAUDE_PLUGIN_ROOT}` for script paths.

## Cursor → Claude Code substitutions (applied everywhere)

| Cursor | Ours |
|---|---|
| Task tool / `subagent_type: generalPurpose` | Agent tool / `general-purpose` |
| `subagent_type: "poteto-agent"`, `"Comment Sicko"` | plugin agents `poteto-agent`, `comment-sicko` |
| `readonly: true` (strips MCP) | read-only agent types / restricted tools; "don't write" posture where MCP is needed |
| `environment: "cloud"` | background agents + `isolation: "worktree"` (local); remote isolation where available |
| `AskQuestion` | `AskUserQuestion` |
| Cursor `/loop` | Claude Code `/loop` skill; watch-pr under background Bash as the event wake |
| `/goal` | standing-orders file re-read at every tick |
| `~/.cursor/rules/pstack-models.mdc` | `~/.claude/pstack-models.md` + CLAUDE.md `@` import |
| `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.claude/projects/<slug>/*.jsonl` |
| built-in `create-skill` | skill-authoring guidance ported into authoring-a-skill playbook |
| cursor-team-kit `deslop` / `control-cli` / `control-ui` | referenced directly (cursor-team-kit plugin exists for Claude Code and is installed) |
| Bugbot | CodeRabbit + Claude review; configurable detection tokens in watch-pr |
| Graphite `gt` | `gh` only (see Phase 5) |
| mode/icon/color/reminder frontmatter | SessionStart hook mandate |

Dropped, with rationale recorded in NOTICE.md: benny (Slack automations), make-bot-ui
(Cursor routines/webhook runtime), Cursor-built-in-babysit caveats, docs/guide (Cursor-UI
tutorial; replaced by our README).

## Phases

Each phase lands independently; verify before advancing (sequence-verifiable-units).

### Phase 0 — Scaffold
Marketplace + plugin manifests, LICENSE/NOTICE (MIT, Lauren Tan attribution, upstream SHA pin),
README stub, empty skill tree.
→ verify: plugin installs locally from this repo; `claude` lists it without errors.

### Phase 1 — Principles + prose layer
All 21 `principle-*` skills (content near-verbatim, `user-invocable: false`), plus
`unslop`, `technical-writing`, `bro`, `tdd`, `typescript-best-practices` (+ references).
→ verify: each skill invocable by name; principle leaves absent from the user picker.

### Phase 2 — Router core
`poteto-mode` SKILL.md (substitutions applied; Claude model roles; deslop/control
references retargeted), all 23 playbooks + `references/bugbot-triage.md` (renamed
review-bot-triage, CodeRabbit + Claude review patterns), `poteto-agent`, `comment-sicko`,
effort-pinned lane agents, SessionStart sticky hook with cross-platform runner (.cmd polyglot).
Graphite-dependent playbook text marked TODO(phase-5), not silently rewritten.
→ verify: fresh session auto-routes a non-trivial task into poteto-mode; todo list carries
playbook steps verbatim; a trivial turn does not trigger.

### Phase 3 — Multi-agent skills + model config
`setup-pstack` (detects Claude models, writes `~/.claude/pstack-models.md`, wires the
CLAUDE.md import), then `how`, `why`, `architect`, `arena`, `swarm`, `interrogate`,
`reflect`, `recall`, `blast-radius`, `figure-it-out`, `teach`, `no-comments`,
`show-me-your-work` (+ log.sh), `automate-me` — transcript paths, MCP discovery, and
panel definitions all substituted.
→ verify: `/how` on a real repo fans out explorers and synthesizes; `/interrogate` on a
diff runs the 3-lane panel and produces lead judgment; `/setup-pstack` round-trips config.

### Phase 4 — Scripts
Vendor `scripts/` under poteto-mode: bootstrap.ts (bun, Windows paths), check-plan.mjs
(lane rule rewritten to "Ten lanes on `sonnet-5` at the PR head"), watch-pr (bot tokens →
CodeRabbit + Claude review + configurable; keep NDJSON contract and exit codes), orch
(frontier rewritten from `gt` to a `gh` base/head-graph implementation), worktree-audit
rewritten portable (bash for Git Bash on Windows; GNU stat/date; `~/.claude/projects`
transcript scan; drop xcrun/simulators).
→ verify: `bun test` and `tsc --noEmit` green on Windows; watch-pr live against a test PR
reaches a correct terminal state; orch frontier matches a hand-built PR chain.

### Phase 5 — Autonomous tier, gh-only, merge-ready ceiling
- **babysit**: gh-only; after opening/pushing, posts `@claude review` on the PR
  automatically and treats both bots' threads via the triage rubric; drives conflicts →
  threads → CI to merge-ready; never merges.
- **shipping** → renamed intent: independent per-PR verification (fresh non-author agent,
  live surface, verdict posted on the PR) then **report merge-ready and stop**. No
  merge-when-ready arming. patch-id staleness rule kept.
- **autopilot-full/stack** → collapsed into one gh-only "autopilot" playbook: one owner
  per PR (interrogate + blast-radius + deslop + no-comments in the owner loop), root
  verification of each head, ceiling at merge-ready; dependent chains as gh base-branch
  chains with retarget-on-merge.
- **orchestrate / autonomous-run / multi-phase-plan**: orch store + drains kept; frontier
  from Phase-4 gh implementation; check-plan template updated to match.
→ verify: end-to-end on a sandbox repo — owner builds a PR, babysit gets CI green and both
bot reviews triaged, shipping verifier posts a verdict, run halts at merge-ready with the
user's merge as the only remaining click.

### Phase 6 — Verification skills, eval, docs
`create-verification-skill` + `maintain-verification-skill` (paths → `.claude/skills/`),
eval playbook (transcript paths), README (install, usage, what changed vs upstream, model
map), UPSTREAM.md (sync procedure pinned to the upstream SHA).
→ verify: create-verification-skill generates and self-proves a verify skill on a sample
app; README instructions reproduce a clean install.

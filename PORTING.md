# Porting spec: Cursor pstack → Claude Code

Ground rules for every ported file. Applies to skills, playbooks, references, and agents.
Upstream wording stays verbatim wherever no substitution applies — do not rewrite, tighten,
or improve prose. Every changed line must trace to a rule below.

Upstream source: cursor/plugins `pstack/` at commit `397c8660da6d3d873a91e18c2ca2f22cac1f0ac1` (v0.14.5).

## Frontmatter

- Keep `name` and `description` (adapt Cursor-specific wording inside the description per the
  substitutions below).
- **Drop `disable-model-invocation` everywhere.** In Claude Code it blocks Skill-tool invocation,
  which would break poteto-mode's routing. Skills stay reachable both by the router and as
  `/pstack:<name>`.
- **Principle skills** (`principle-*`): add `user-invocable: false` (hidden from the slash picker;
  still readable and Skill-invocable).
- **Drop** `mode:`, `icon:`, `color:`, `reminder:` (Cursor sticky-mode features; the SessionStart
  hook replaces them).
- Agents (`agents/*.md`): keep `name`/`description`; drop `is_background`; add
  `model` and effort pinning per the model map when the agent implies one.

## Model map

| Upstream slug / role | Port |
|---|---|
| `grok-4.6-fast-xhigh` (fast mechanical code: feature/refactoring delegates, swarm workers, live lanes, how explorers, why investigators) | `sonnet-5` at high effort — spawn via the `pstack-sonnet-high` agent |
| `gpt-5.6-sol-max` (precisely specified code: bug-fix, perf, hillclimb delegates) | `opus-5` at max effort — `pstack-opus-max` |
| `claude-fable-5-thinking-max` (judgment, prose, hardest changes, synthesizers, leads) | `fable-5` at max effort — `pstack-fable-max` |
| `claude-opus-5-thinking-xhigh` (panel member) | `opus-5` at high effort — `pstack-opus-high` |
| Default review panel (interrogate reviewers, how critics, arena runners, architect runners) | three lanes: `fable-5` max, `opus-5` high, `sonnet-5` high |
| "a different model family" (judges, verifiers, cross-model review) | "a different model tier than the one that did the work" |
| `inherit-parent` / `auto` | unchanged meaning: omit the model override on the Agent call |

Config file: `~/.claude/pstack-models.md` replaces `~/.cursor/rules/pstack-models.mdc`.
Written by `/setup-pstack`, imported from the user's `~/.claude/CLAUDE.md` via `@` import.
Role lines override the defaults above; a role with no line keeps its default.

## Tool and platform substitutions

| Cursor | Port |
|---|---|
| `Task` tool | `Agent` tool |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` |
| `subagent_type: "poteto-agent"` / `"Comment Sicko"` | `subagent_type: "pstack:poteto-agent"` / `"pstack:comment-sicko"` |
| `readonly: true` | a read-only agent type (e.g. Explore) or an explicit no-writes instruction. Preserve upstream's nuance: read-only agent types can lack MCP access, so where a worker needs MCPs (why investigators, reflect reviewers), use a writing-capable type with "do not write" as posture |
| `run_in_background: true` | drop the parameter; Claude Code subagents already run in the background. Keep "spawn all N in one message" |
| `environment: "cloud"` / cloud agents / cloud VMs | background agents with `isolation: "worktree"`; `isolation: "remote"` where the account has cloud access. Keep the placement logic (local only when the task needs the user's machine) |
| `cloud_base_branch` | the worktree/branch named in the agent prompt |
| `AskQuestion` (`allow_multiple`) | `AskUserQuestion` (`multiSelect`) |
| Cursor `/loop` ("dynamic mode") | Claude Code's `/loop` skill; dynamic mode = `/loop` with no interval (self-paced). Watcher processes run under background Bash as the event wake |
| `/goal` | a standing-orders file re-read at every tick (orchestrate store `preferences.md`) |
| todolist | the todo list (TodoWrite); contract unchanged: playbook steps copied verbatim, `skip: <reason>` |
| `~/.cursor/rules/pstack-models.mdc` | `~/.claude/pstack-models.md` |
| `~/.cursor/projects/<slug>/agent-transcripts/<id>/<id>.jsonl` | `~/.claude/projects/<slug>/*.jsonl` (slug = workspace path with separators replaced by `-`); same never-glob-other-projects rule |
| `.cursor/skills/` / `~/.cursor/skills/` | `.claude/skills/` / `~/.claude/skills/` |
| `.cursor/worktrees/<repo>/` | the repo's worktree locations as listed by `git worktree list` |
| Cursor built-in `create-skill` | the authoring-a-skill playbook's own guidance (validation rules inlined there) |
| Cursor built-in `/babysit` shadowing caveat | delete (no Claude Code built-in to shadow) |
| `cursor-team-kit` `deslop` / `control-cli` / `control-ui` | same skills from the Claude Code `cursor-team-kit` plugin: `cursor-team-kit:deslop`, `cursor-team-kit:control-cli`, `cursor-team-kit:control-ui` |
| Bugbot / "agentic security review" | "review bots" = CodeRabbit (fires automatically) and Claude review (triggered by commenting `@claude review` on the PR). `references/bugbot-triage.md` → `references/review-bot-triage.md` |
| Cursor dashboard (cloud-agent status) | probe by side effects only (ledger, pushed branches, `gh`); `/tasks` for local task status |
| "restart Cursor" / context compaction | "restart Claude Code" / context compaction (same trigger; `--resume` for pickup) |
| Cursor image-generation tool (teach) | mermaid diagrams only; drop the whiteboard-image alternative |
| Graphite `gt` commands, merge-when-ready, Graphite UI | Phase 5 rework: plain `gh`; autonomous ceiling is merge-ready, never merging. Until a file gets its Phase 5 pass, mark Graphite-dependent passages with `<!-- TODO(phase-5): graphite -->` rather than silently rewriting |

## Path and naming conventions

- Skill cross-references stay relative within the plugin (`skills/<name>/SKILL.md`,
  `playbooks/<name>.md`); slash references become `/pstack:<name>`.
- Scripts are invoked via `${CLAUDE_PLUGIN_ROOT}/skills/poteto-mode/scripts/...`.
- `make-bot-ui` and `automations/benny` are not ported (see NOTICE.md).

## What must survive verbatim

The verification doctrine: "Tests alone are not sufficient verification…", "CI green is an
input to a verdict, not a verdict", "Inconclusive or wrong-surface is not a pass", the
`git patch-id` staleness rule, state-then-wait, untrusted-data handling for bot comments,
the unslop catalog, the long-dash ban, principle wording, and every numbered playbook step
(steps change only where a substitution applies; never dropped).

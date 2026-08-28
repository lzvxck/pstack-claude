# poteto-mode

Claude Code port of [pstack](https://github.com/cursor/plugins/tree/main/pstack) by Lauren Tan
(@poteto). pstack turns a short request into a complete engineering workflow: a router mode,
22 task playbooks, 23 workflow skills, 21 engineering principles, adversarial review panels,
and autonomous runs that stop at verified merge-ready.

This port is Claude-only (Sonnet 5 / Opus 5 / Fable 5 with effort tiers), replaces Graphite
with plain `gh`, and never merges a PR: the autonomous tier's ceiling is an independently
verified merge-ready report, and the merge is always your click.

## Install

```bash
/plugin marketplace add https://github.com/lzvxck/pstack-claude
/plugin install pstack@poteto-mode
```

Then, in a new session:

1. Run `/pstack:setup-pstack` to write your per-role model config (`~/.claude/pstack-models.md`).
2. Use `/pstack:poteto-mode` at the start of any task that needs rigor. A SessionStart hook
   also routes non-trivial engineering tasks into it automatically; opt out any time by saying so.

## What you get

- **`/pstack:poteto-mode`** routes your request to one of 22 playbooks (bug fix, feature,
  refactoring, perf, hillclimb, forensics, prototype, visual parity, eval, babysit, shipping,
  autopilot, orchestrate, and more), copies the playbook's steps into the todo list verbatim,
  and demands runtime evidence before anything is called done.
- **Understanding skills**: `/pstack:how` (parallel explorers explain a subsystem),
  `/pstack:why` (evidence-cited design rationale from git, trackers, docs, and whatever MCPs
  you have), `/pstack:teach`, `/pstack:recall`, `/pstack:blast-radius`.
- **Design and review skills**: `/pstack:architect`, `/pstack:arena` (N candidates, blind
  cross-tier judge, graft the winners), `/pstack:swarm`, `/pstack:interrogate` (multi-lane
  adversarial diff review).
- **Discipline skills**: `/pstack:unslop`, `/pstack:technical-writing`, `/pstack:no-comments`
  (spawns the Comment Sicko agent), `/pstack:tdd`, `/pstack:show-me-your-work` (auditable
  decision trail), `/pstack:create-verification-skill`.
- **21 principle skills** the router indexes and cites against real decisions.
- **Agents**: `poteto-agent`, `comment-sicko`, and four effort-pinned model lanes
  (`pstack-sonnet-high`, `pstack-opus-high`, `pstack-opus-max`, `pstack-fable-max`).
- **Scripts** (bun): `watch-pr` (PR merge-readiness watcher, exit-code verdicts, CodeRabbit
  and Claude-review thread stamping), `orch` (orchestration store CLI with a gh-based chain
  frontier), `check-plan.mjs` (plan lint), `worktree-audit.sh` (safe worktree prune audit).

## Model roles

| Role | Default lane |
|---|---|
| Fast mechanical code (feature/refactoring delegates, swarm workers, explorers, live lanes) | `pstack-sonnet-high` |
| Precisely specified code (bug-fix, perf, hillclimb delegates) | `pstack-opus-max` |
| Judgment, prose, synthesis, hardest changes | `pstack-fable-max` |
| Review panels (interrogate, arena, architect, how critics) | `pstack-fable-max` + `pstack-opus-high` + `pstack-sonnet-high` |

`/pstack:setup-pstack` overrides any role. Judges and verifiers run on a different model tier
than the worker they check.

## Review bots

Babysit and the autopilot tier triage two bots per `review-bot-triage.md`: CodeRabbit (fires
automatically) and Claude review (the workflow posts `@claude review` on each PR it opens).
Other bots can be added via the `PSTACK_REVIEW_BOTS` env var (comma-separated author logins)
read by `watch-pr`.

## Requirements

- Claude Code with this plugin installed; the `cursor-team-kit` plugin for `deslop`,
  `control-cli`, and `control-ui` (referenced by several playbooks).
- `bun` for the scripts, `gh` (authenticated) for anything PR-shaped.
- `jq` and `rg` for `worktree-audit.sh` (present in Git Bash environments).

## What changed from upstream

See `NOTICE.md` for the full list. The short version: Claude-only model routing with effort
lanes, sticky mode via a SessionStart hook instead of Cursor mode frontmatter, `gh` instead
of Graphite, merge-ready instead of merging, CodeRabbit/Claude-review instead of Bugbot,
Claude Code transcript and skill paths, and the benny Slack automation pack plus the
`make-bot-ui` skill dropped. The porting spec lives in `PORTING.md`; the sync procedure in
`UPSTREAM.md`.

## License

MIT. Original work by Lauren Tan; see `LICENSE` and `NOTICE.md`.

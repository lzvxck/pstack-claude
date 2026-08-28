# Notices and attribution

This repository is a Claude Code port of **pstack** by Lauren Tan (@poteto).

| Component | Origin | License |
|---|---|---|
| All skills, playbooks, principles, agents, scripts under `plugins/pstack/` | [cursor/plugins](https://github.com/cursor/plugins) `pstack/` at upstream v0.14.5, commit `397c8660da6d3d873a91e18c2ca2f22cac1f0ac1` | MIT (Lauren Tan) — see `LICENSE` |
| `plugins/pstack/hooks/run-hook.cmd` polyglot pattern | superpowers plugin (MIT), via the pstack-claude port by Michael Denyer (MIT) | MIT |
| Cursor-to-Claude substitution decisions | Informed by [pstack-claude](https://github.com/michael-denyer/pstack-claude) and [open-pstack](https://github.com/ericlitman/open-pstack) changelogs (both MIT); no file content copied except the hook runner pattern above | MIT |

## Deliberate departures from upstream

- **Claude-only model routing.** Cursor's cross-vendor panel (Fable / Sol / Grok / Opus) becomes Sonnet 5 / Opus 5 / Fable 5 with effort tiers; "different model family" judge rules become "different tier than the worker".
- **No Graphite.** Stacked-PR mechanics run on plain `gh`; the autonomous tier stops at merge-ready and never merges.
- **Review bots.** Bugbot triage is re-targeted to CodeRabbit and Claude review (`@claude review`).
- **Dropped**: `automations/benny` (Cursor Slack automations runtime), the `make-bot-ui` skill (Cursor webhook-routine runtime), Cursor's `docs/guide` tutorial (Cursor-UI-bound), and the `mode:`/`icon`/`color`/`reminder` sticky-mode frontmatter (replaced by a SessionStart hook).

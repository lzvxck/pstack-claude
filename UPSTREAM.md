# Upstream sync

Upstream is [cursor/plugins](https://github.com/cursor/plugins), directory `pstack/`.

Current sync: **v0.14.5**, commit `397c8660da6d3d873a91e18c2ca2f22cac1f0ac1` (2026-08-28).

## Procedure

1. Clone or fetch upstream and diff `pstack/` between the pinned commit above and the new
   target commit. Review the commits in order, not the squashed diff.
2. For each changed file, re-apply the change here through `PORTING.md`: keep upstream
   wording verbatim, apply only the substitutions the spec names. Files this port rewrote
   (babysit, shipping, autopilot, orchestrate, multi-phase-plan, opening-a-pr,
   worktree-cleanup, review-bot-triage, setup-pstack, the scripts) need a manual merge of
   upstream's intent into the rewritten shape rather than a mechanical re-port.
3. Files this port dropped stay dropped (see NOTICE.md) unless a decision changes.
4. Run the verification sweep: `bun test orch watch-pr` and the typecheck in
   `plugins/pstack/skills/poteto-mode/scripts/`, plus a grep for Cursor-isms over
   `plugins/pstack/**/*.md` (model slugs, `.cursor/` paths, Task-tool params, Bugbot,
   Graphite, `/goal`, `agent-transcripts`).
5. Update the pinned commit and version here and in NOTICE.md, and bump the plugin version
   in both manifests.

New pstack behavior belongs upstream first when possible; this port tracks, it does not fork
the ideas.

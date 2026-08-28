### Worktree cleanup

**You own the disk and the safety gate.** Prune merged or abandoned git worktrees to reclaim space. Deletion is irreversible, so every step guards against deleting something in use or holding uncommitted work.

1. Snapshot and audit. Record `df -h .`, then run `${CLAUDE_PLUGIN_ROOT}/skills/poteto-mode/scripts/worktree-audit.sh` (principle-build-the-lever). It reads paths from `git worktree list`, never hand-typed, since a hand-typed `myrepo-worktrees/x` misses one that lives under another root (principle-encode-lessons-in-structure). It classifies each worktree by size, age, merge state, uncommitted work, PR state, and the newest session that touched it, then suggests a bucket. The transcript scan is slow, so background it.
2. The bucket is advice, not permission. The user's active and kept sessions are the real artifact (principle-prove-it-works). Get that set from the user and cross-check every candidate. The lever has marked `safe` a worktree the user still had a session on, so the user's set wins.
3. Verify usage before deleting. For every `verify-recent-chat` row, or anything you doubt, fan subagents out to read the session transcripts under `~/.claude/projects/<slug>/` and report whether the session is ongoing and which worktrees it touches (principle-guard-the-context-window, transcripts are bulk). A session spawns arena and repro trees into sibling worktrees via background subagents, and those are in use even when their names never surface to the user.
4. Pause on irreversible loss. `wip:N` is N tracked uncommitted edits. Show the diff and get a decision first, since removing a clean worktree is recoverable from its branch but uncommitted work is gone. `scratch:N` is untracked throwaway, safe to drop, but name the files. Per Autonomy, clean and merged and not-in-use proceeds; `wip` and in-use pause.
5. Prune the confirmed set. Per path, `git worktree remove --force <path>`; if the dir survives on ignored build artifacts, `rm -rf` it, then `git worktree prune`. Branch refs survive, so no commits are lost. Confirm with `df -h .` and re-list.
6. Other reclaimers when needed: package caches (npm, pnpm, yarn, bun, pip, uv) and stale build output dirs. Clear only caches the user has not said to keep.

This is the one playbook that deletes user state with no code review to catch a slip, so the gates above are the review.

**Reply:** `df -h .` before and after with space reclaimed, the worktrees pruned, and a one-line reason for each held back (in-use by which session, or uncommitted work).

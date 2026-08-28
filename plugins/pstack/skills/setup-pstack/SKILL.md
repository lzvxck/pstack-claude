---
name: setup-pstack
description: Configure which models pstack uses per role. Detects your available models and writes an always-applied config file that overrides the skill defaults. Use for /pstack:setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Write `~/.claude/pstack-models.md`, an always-loaded config file that sets pstack's model per role, and make sure `~/.claude/CLAUDE.md` imports it. The skills read it and fall back to their inline defaults when a line is absent, so this is an override layer, not a requirement.

## Steps

### 1. Detect available models

The models you can pass to an `Agent` subagent in this session (the Agent tool's `model` values) are the dependable source. pstack also ships effort-pinned lanes as plugin agents, and lane names are the preferred role values because they carry both model and effort: `pstack-sonnet-high` (Sonnet 5, high effort), `pstack-opus-high` (Opus 5, high effort), `pstack-opus-max` (Opus 5, max effort), `pstack-fable-max` (Fable 5, max effort). If you cannot confirm the user is entitled to a model, ask rather than write it. The aliases `inherit-parent` and `auto` are always valid even though they are not detected values.

### 2. Load current state

The default role-to-model mapping is the file shape shown in step 5 below. If `~/.claude/pstack-models.md` already exists, read it and treat its values as the current choices. Otherwise start from those defaults.

### 3. Map and confirm

Show every role with its current value, marking any value not in the detected set as needing a choice. Ask whether to accept as-is or change specific roles, offering the shipped lanes plus `inherit-parent` and `auto` (both mean: this role runs on the parent chat model, which is how default-model users stay on their default) as the options. Prefer AskUserQuestion over free text. For panel roles (how critics, arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model tier differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real value written must be a shipped lane or a confirmed Agent model; `inherit-parent` and `auto` always pass. If a chosen value is not available, stop and ask again. A config pointing at a model the user cannot use breaks every delegation that reads it.

### 5. Write the config

Write `~/.claude/pstack-models.md` with one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Then read `~/.claude/CLAUDE.md` and, if it does not already import the file, append the line `@pstack-models.md` so the config loads in every session. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the Agent `model` override). Alias entries in a panel list still count toward its fan-out.
feature, refactoring: pstack-sonnet-high
bug-fix: pstack-opus-max
perf-issue: pstack-opus-max
hillclimb: pstack-opus-max
judgment and prose: pstack-fable-max
hardest tasks: pstack-fable-max
how explorer: pstack-sonnet-high
how explainer: pstack-fable-max
how critics: pstack-fable-max, pstack-opus-high, pstack-sonnet-high
why investigators: pstack-sonnet-high
why synthesizer: pstack-fable-max
reflect tooling: pstack-opus-max
reflect judgment, divergent, synthesizer: pstack-fable-max
arena runners: pstack-fable-max, pstack-opus-high, pstack-sonnet-high
arena cross-judge pool: pstack-fable-max, pstack-opus-high, pstack-sonnet-high
swarm workers: pstack-sonnet-high
architect runners: pstack-fable-max, pstack-opus-high, pstack-sonnet-high
interrogate reviewers: pstack-fable-max, pstack-opus-max, pstack-sonnet-high
```

### 6. Confirm

Tell the user the config was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /pstack:create-verification-skill." On yes, invoke `/pstack:create-verification-skill`. On no, move on without pushing.

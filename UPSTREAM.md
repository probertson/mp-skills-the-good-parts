# Upstream

The skills in this plugin were copied from [mattpocock/skills](https://github.com/mattpocock/skills) and then changed. This file is the record an agent needs to bring in later upstream changes without losing ours.

## Baseline

Copied from upstream commit `4ae2317ddb22b046e26455e4f9c9b3aaaf62212d` (2026-10-31, plugin version 1.3.2).

The first commit in this repo is that copy, unchanged apart from the files listed under "Not copied". Every later commit is one of our changes, so `git diff` from that first commit shows everything we've changed.

## Where each skill came from

| Here | Upstream |
|---|---|
| `skills/codebase-design/` | `skills/engineering/codebase-design/` |
| `skills/domain-modeling/` | `skills/engineering/domain-modeling/` |
| `skills/grill-me/` | `skills/productivity/grill-me/` |
| `skills/grilling/` | `skills/productivity/grilling/` |
| `skills/improve-codebase-architecture/` | `skills/engineering/improve-codebase-architecture/` |
| `skills/prototype/` | `skills/engineering/prototype/` |
| `skills/research/` | `skills/engineering/research/` |
| `skills/wayfinder/` | `skills/engineering/wayfinder/` |

`LICENSE` is upstream's, unchanged.

### Not copied

- Every skill's `agents/openai.yaml` (OpenAI agent metadata; not used by Claude Code).
- `skills/engineering/setup-matt-pocock-skills/`. Its "Wayfinding operations" for local markdown files were folded into `wayfinder` (see Changes below); nothing else here needs it.

## Bringing in upstream changes

1. In a clone of upstream, diff from the last synced commit (the Baseline above, or the latest entry under "Sync history") to the new one, limited to the upstream paths in the table above.
2. For each upstream change, decide whether we want it. If we do, apply it to our copy, keeping every change listed below intact. Where an upstream change conflicts with one of ours, the intent recorded below wins unless we decide otherwise.
3. Check for new cross-skill calls: if an upstream skill starts calling a skill we don't have, either copy that skill too (and add it to the table) or adapt the call.
4. Add an entry to "Sync history".

## Changes from upstream

Each entry says what we changed and why, so it can be re-applied to a new upstream version whose wording has moved.

## Sync history

- 2026-11-03: initial copy at `4ae2317`.

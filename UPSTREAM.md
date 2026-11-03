# Upstream

The skills in this plugin were copied from [mattpocock/skills](https://github.com/mattpocock/skills) and then changed. This file is the record an agent needs to bring in later upstream changes without losing ours.

## Baseline

Copied from upstream commit `4ae2317ddb22b046e26455e4f9c9b3aaaf62212d` (2026-10-31, plugin version 1.3.2).

The first commit in this repo is that copy, unchanged apart from the files listed under "Not copied", plus this plugin's own files (`README.md`, `UPSTREAM.md`, `.claude-plugin/`, `.gitignore`). Later commits are either our changes or syncs from upstream (see "Sync history"), so `git diff` from that first commit shows how our copy differs from the baseline.

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

### 1. Cross-skill calls use plugin-qualified names

**What:** Every place one skill calls another uses its `mp-good:` name: "Call the Skill tool with "mp-good:grilling"", "the `/mp-good:codebase-design` skill", and so on.

**Why:** Upstream's `mattpocock-skills` plugin may be installed alongside this one, with skills of the same names. A bare name like "grilling" could load upstream's version, and our changes would silently not apply.

**When syncing:** Qualify any new cross-skill call that upstream adds.

### 2. Wayfinder keeps its map in markdown files, not an issue tracker

**What:** `wayfinder` no longer uses an issue tracker or the per-repo `docs/agents/issue-tracker.md` that upstream's `setup-matt-pocock-skills` writes. A map is a folder with `map.md` and a `tickets/` subfolder of `NN-<slug>.md` files. Each ticket has `Type:`, `Status:` (`open` / `claimed` / `resolved` / `out-of-scope`) and `Blocked by:` lines, and the answer is appended under `## Answer`. This is based on upstream's local-markdown "Wayfinding operations" (in `setup-matt-pocock-skills/issue-tracker-local.md`), folded into the skill. Where the folder goes is decided at charting time: a location named in the repo's `CLAUDE.md` / `AGENTS.md`, otherwise ask the user (suggesting `.scratch/<effort>/` when the repo has one; a temp folder is allowed, with a warning). Research findings go in the map's `research/NN-<slug>.md`, linked from the ticket, instead of upstream's pushed throwaway `research/<name>` git branch.

**Why:** So the skill works in any repo without a setup step. Where planning files belong varies by repo, so the skill asks rather than assuming.

**When syncing:** Translate upstream changes to tracker operations (map, child tickets, labels, claiming, blocking, frontier, resolving) into their file equivalents. Check upstream's `issue-tracker-local.md` "Wayfinding operations" for changes worth bringing in. Ignore changes to the GitHub/GitLab tracker docs.

### 3. Wayfinder doesn't assume an effort outgrows one session

**What:** Removed upstream's premise that a map exists because the work is "more than one agent session can hold". The description and opening say the work is too big or foggy to decide in one go. Tickets are sized to one decision, not "one 100K token agent session". The "never resolve more than one ticket per session" rule is gone, and "Work through the map" loops back to the next frontier ticket until the user stops or the frontier is empty. Charting ends by printing the map path and offering to start on the first frontier ticket, instead of "charting is one session's work". The "you don't need a map" exit now depends on whether there's real fog, not on session size.

**Why:** Upstream's view of agent context size is more conservative than ours. We don't want agents stopping or suggesting compaction between tickets. The map is valuable for structuring decisions, not for working around context limits. Maps can still be resumed in a later session or worked in parallel, so wording about sessions in general (claiming, concurrent edits, "loaded once per session") stays.

**When syncing:** Watch for new wording that ties tickets, steps or stopping points to session or context size, and drop it. Fresh-context subagents used for parallelism (research, fact-finding) are fine to keep.

## Sync history

- 2026-11-03: initial copy at `4ae2317`.

# Upstream

The skills in this plugin were copied from [mattpocock/skills](https://github.com/mattpocock/skills) and then changed. This file is the record an agent needs to bring in later upstream changes without losing ours.

## Baseline

Copied from upstream commit `4ae2317ddb22b046e26455e4f9c9b3aaaf62212d` (2026-10-31, plugin version 1.3.2).

The first commit in this repo is that copy, unchanged apart from the files listed under "Not copied", plus this plugin's own files (`README.md`, `UPSTREAM.md`, `.claude-plugin/`, `.gitignore`). Later commits are either our changes or syncs from upstream (see "Sync history"), so `git diff` from that first commit shows how our copy differs from the baseline. Skills added later were each copied unchanged in a commit of their own before being changed (see "Sync history"), so for those, diff from that copy commit.

## Where each skill came from

| Here | Upstream |
|---|---|
| `skills/codebase-design/` | `skills/engineering/codebase-design/` |
| `skills/domain-modeling/` | `skills/engineering/domain-modeling/` |
| `skills/grill-me/` | `skills/productivity/grill-me/` |
| `skills/grilling/` | `skills/productivity/grilling/` |
| `skills/implement-spec/` | `skills/engineering/implement-spec/` |
| `skills/improve-codebase-architecture/` | `skills/engineering/improve-codebase-architecture/` |
| `skills/prototype/` | `skills/engineering/prototype/` |
| `skills/research/` | `skills/engineering/research/` |
| `skills/tdd/` | `skills/engineering/tdd/` |
| `skills/wayfinder/` | `skills/engineering/wayfinder/` |

`LICENSE` is upstream's, unchanged.

### Not copied

- Every skill's `agents/openai.yaml` (OpenAI agent metadata; not used by Claude Code).
- `skills/engineering/setup-matt-pocock-skills/`. Its "Wayfinding operations" for local markdown files were folded into `wayfinder` (see Changes below). `implement-spec` reads the `docs/agents/issue-tracker.md` it writes when a repo already has one, but doesn't require it.

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

**When syncing:** Qualify any new cross-skill call that upstream adds. (`tdd` and `implement-spec` got the same treatment when they were added.)

### 2. Wayfinder keeps its map in markdown files, not an issue tracker

**What:** `wayfinder` no longer uses an issue tracker or the per-repo `docs/agents/issue-tracker.md` that upstream's `setup-matt-pocock-skills` writes. A map is a folder with `map.md` and a `tickets/` subfolder of `NN-<slug>.md` files. Each ticket has `Type:`, `Status:` (`open` / `claimed` / `resolved` / `out-of-scope`) and `Blocked by:` lines, and the answer is appended under `## Answer`. This is based on upstream's local-markdown "Wayfinding operations" (in `setup-matt-pocock-skills/issue-tracker-local.md`), folded into the skill. Where the folder goes is decided at charting time: a location named in the repo's `CLAUDE.md` / `AGENTS.md`, otherwise ask the user (suggesting `.scratch/<effort>/` when the repo has one; a temp folder is allowed, with a warning). Research findings go in the map's `research/NN-<slug>.md`, linked from the ticket, instead of upstream's pushed throwaway `research/<name>` git branch.

**Why:** So the skill works in any repo without a setup step. Where planning files belong varies by repo, so the skill asks rather than assuming.

**When syncing:** Translate upstream changes to tracker operations (map, child tickets, labels, claiming, blocking, frontier, resolving) into their file equivalents. Check upstream's `issue-tracker-local.md` "Wayfinding operations" for changes worth bringing in. Ignore changes to the GitHub/GitLab tracker docs.

### 3. Wayfinder doesn't assume an effort outgrows one session

**What:** Removed upstream's premise that a map exists because the work is "more than one agent session can hold". The description and opening say the work is too big or foggy to decide in one go. Tickets are sized to one decision, not "one 100K token agent session". The "never resolve more than one ticket per session" rule is gone, and "Work through the map" loops back to the next frontier ticket until the user stops or the frontier is empty. Charting ends by printing the map path and offering to start on the first frontier ticket, instead of "charting is one session's work". The "you don't need a map" exit now depends on whether there's real fog, not on session size.

**Why:** Upstream's view of agent context size is more conservative than ours. We don't want agents stopping or suggesting compaction between tickets. The map is valuable for structuring decisions, not for working around context limits. Maps can still be resumed in a later session or worked in parallel, so wording about sessions in general (claiming, concurrent edits, "loaded once per session") stays.

**When syncing:** Watch for new wording that ties tickets, steps or stopping points to session or context size, and drop it. Fresh-context subagents used for parallelism (research, fact-finding) are fine to keep.

### 4. Grilling asks one question at a time, after the facts are in

**What:** `grilling` no longer asks "the whole frontier in one round" as a numbered list. It asks one question per message, the one on the frontier whose answer unblocks the most, with a recommended answer, and the question is the last thing in the message. It no longer asks questions while a fact-finding lookup is still running ("Don't block on it"). It finishes the lookup, recomputes the frontier, then asks. The design tree, frontier, recommended answers, "yes accepts the recommendation", "finding facts is your job", and "confirm shared understanding before acting" are kept. This also applies to `grill-me`, `improve-codebase-architecture` and `wayfinder`, which all use `grilling`; `wayfinder`'s breadth-first charting step notes that breadth-first sets the order of questions, not how many are asked at once.

**Why:** Several questions at once are hard to follow, especially when the user replies to one with a question or a longer answer of their own. Questions asked while a lookup is running often come back reframed by its result, which wastes the user's attention.

**When syncing:** Keep one question per message and facts-before-questions. Bring in other upstream improvements to how the tree or frontier is worked, translating any "round" or batch wording into single questions.

### 5. implement-spec works without the setup skill

**What:** `implement-spec` no longer tells the user to run `/setup-matt-pocock-skills` when no tracker is configured. Its opening says the spec may be a `.scratch/<feature>/` folder or a tracker issue. If the repo has a `docs/agents/issue-tracker.md` (written by upstream's setup skill), it follows that for reading the spec and tickets, blocking, and closing work. If not, or if the doc doesn't say how something works for a spec's tickets, it asks the user. Its description says "a spec and its tickets, on one integration branch, with a subagent per ticket" instead of "the result of /to-spec and /to-tickets", since we don't have those skills.

**Why:** The repos we use it in already have upstream's tracker docs (local markdown in `.scratch/`, or GitHub issues), so following them works; we just don't want the skill to require the setup step or the spec-writing skills.

**When syncing:** Bring in upstream changes to the implementation flow. Where upstream adds new references to `/to-spec`, `/to-tickets` or the setup skill, make them optional in the same way.

### 6. tdd and implement-spec use our reviewer

**What:** `implement-spec`'s final review step (step 8) reviews the integration branch with `probertson-review:self-review` (from the `probertson-review` plugin in [probertson/skills](https://github.com/probertson/skills)), in a checkout with no uncommitted changes, handing it the spec to review against and the base branch to diff from, instead of upstream's `code-review`. Step 1 checks that the skill is installed and stops before any work if it isn't. `tdd`'s note that refactoring belongs to the review stage points at the same skill.

**Why:** We prefer our own review standards to upstream's `code-review`. The step stays, rather than relying on per-commit reviews, because it reviews the whole integration branch, which catches problems that only show up across tickets.

**When syncing:** Keep the review step pointed at `probertson-review:self-review`. If upstream changes what the review step does (for example, what happens to its findings), bring that in.

### 7. implement-spec confirms test seams up front

**What:** `implement-spec` has a new step 3, before any implementer starts: for each ticket it proposes the seams under test (the way `tdd` describes) and gets the user's confirmation, one ticket at a time. The confirmed seams are saved to a notes file outside the repo, and every implementer's brief points at it, including the final review-fix implementer. If an implementer reports it needs an unconfirmed seam, that isn't completion: the orchestrator asks the user, adds the answer to the file, and continues that implementer. Later steps were renumbered. `tdd` says that seams handed to it as already confirmed don't need asking again, and that it should stop and report a new seam rather than test there.

**Why:** Upstream `tdd` writes no test at a seam the user hasn't confirmed, but upstream `implement-spec` runs `tdd` in background subagents that can't ask the user, so each one would either stall or break that rule.

**When syncing:** Check whether upstream `tdd` still requires user-confirmed seams and whether upstream `implement-spec` now supplies them. If upstream resolves the conflict its own way, compare it with this one and pick one. Keep step references in `implement-spec` correct if upstream adds or removes steps.

## Sync history

- 2026-11-03: initial copy at `4ae2317`.
- 2026-11-07: added `tdd` and `implement-spec`, also copied at `4ae2317`.

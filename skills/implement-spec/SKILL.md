---
name: implement-spec
description: "Implement a spec and its tickets in code, on one integration branch, with a subagent per ticket."
disable-model-invocation: true
---

You have been provided a spec, such as a `.scratch/<feature>/` folder or an issue on the repo's tracker. This spec should have tickets associated with it, describing how to implement the spec.

If the repo has a `docs/agents/issue-tracker.md`, follow it for how the spec and tickets are stored, how blocking is recorded, and how work is closed. If it doesn't, or it doesn't say how one of these works for a spec's tickets (a "Wayfinding operations" section describes wayfinder maps, not specs), ask the user before starting rather than guessing.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Start every subagent with a fresh context window.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. Check that the `probertson-review:self-review` skill is available (step 7 needs it). If it isn't, stop and tell the user before any work starts. Then read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 5 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

4. Use a fresh **implementer subagent** for each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with "mp-good:tdd" to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

5. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

7. Once all tickets are complete, call the Skill tool with "probertson-review:self-review" on the integration branch, in a checkout with no uncommitted changes, handing it the spec (with its source) to review against and the base branch to diff the integration branch from. Fix all issues raised by the review in a single fresh **implementer subagent**.

8. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

9. Clean up all **implementer subagent** worktrees.

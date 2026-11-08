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

1. Check that the `probertson-review:self-review` skill is available (step 8 needs it). If it isn't, stop and tell the user before any work starts. Then read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Agree the seams with the user. **Implementer subagents** run in the background and can't ask the user anything, but `mp-good:tdd` only writes tests at seams the user has confirmed. So confirm them now: for each ticket, propose the seams under test the way `mp-good:tdd` describes (each with a one-line note on what it catches and what it misses), and get the user's confirmation, one ticket at a time. Save the confirmed seams, per ticket, to a notes file outside the repo (alongside step 2's notes, if any), so every later subagent can be pointed at it. Later tickets may turn out to need seams that don't exist yet; that's expected, and is handled in step 5.

4. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 6 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

5. Use a fresh **implementer subagent** for each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - is pointed at the confirmed-seams file from step 3 in its brief (including implementers started later, in step 7);
   - calls the Skill tool with "mp-good:tdd" to build the ticket, testing only at its ticket's confirmed seams;
   - merges the integration branch tip into its own branch before reporting done

   An implementer that reports it needs a seam that wasn't confirmed has not completed: don't merge its work. Ask the user about the seam, add the answer to the confirmed-seams file, then continue that implementer.

6. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**.

7. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

8. Once all tickets are complete, call the Skill tool with "probertson-review:self-review" on the integration branch, in a checkout with no uncommitted changes, handing it the spec (with its source) to review against and the base branch to diff the integration branch from. Fix all issues raised by the review in a single fresh **implementer subagent**, pointed at the confirmed-seams file like the others; if a fix needs a new seam, handle it as in step 5.

9. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

10. Clean up all **implementer subagent** worktrees.

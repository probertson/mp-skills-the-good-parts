---
name: wayfinder
description: Plan a large, foggy chunk of work (too many open decisions to settle in one go) as a map of decision tickets kept in markdown files, and resolve them one at a time until the way to the destination is clear.
disable-model-invocation: true
---

A loose idea has arrived, too big to decide in one go, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. This skill charts the way as a **map**, a folder of markdown files, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.

The destination varies per effort, and naming it is the first act of charting: it shapes every ticket. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place like a data-structure migration. The map is domain-agnostic: engineering work, course content, whatever fits the shape.

## Plan, don't do

Wayfinder is **planning** by default: each ticket resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.

## Refer by name

Every map and ticket is a markdown file, so it has a **name**: its title, the file's top heading. In prose the human reads (narration, the map's Decisions-so-far), refer to it by that name, never by a bare number, filename, or slug. Fields like `Blocked by:` still use ticket numbers. A wall of `03, 04, 07` is illegible; names read at a glance. The number and path don't vanish; a name wraps a relative link to its file, but they ride _inside_ the name, never stand in for it.

## The Map

The map is a folder holding `map.md`, the canonical artifact, a `tickets/` subfolder with one file per ticket, and a `research/` subfolder for research findings, created when the first one is written.

The map is an **index**, not a store. It lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place, its ticket, so the map never restates it, only gists it and links.

### Where the map lives

Choose the map's folder when charting:

1. If the repo's `CLAUDE.md` or `AGENTS.md` says where wayfinder maps go, use `<that location>/<effort-slug>/`.
2. Otherwise, ask the user. If the repo has a `.scratch/` folder, suggest `.scratch/<effort-slug>/`. A temp folder is fine if the user picks one, but warn them it may be cleared before the map is finished (for example, when the session ends or the machine restarts).

If the folder already exists, ask the user before using it.

Once the map exists, give the user the full path to its `map.md`. That path is how the map is found later.

### The map body

The whole map at low resolution, loaded once per session. Open tickets are **not** listed: they are found by reading the `Status:` lines of the files in `tickets/`.

```markdown
# <map title>

## Destination

<what reaching the end of this map looks like: the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a ticket.>

## Notes

<domain; skills every session should consult; standing preferences for this effort>

## Decisions so far

<!-- the index: one line per closed ticket, enough to judge relevance, then zoom the link for the detail the ticket holds -->

- [<closed ticket title>](tickets/NN-<slug>.md): <one-line gist of the answer>

## Not yet specified

<!-- see "Fog of war": in-scope fog you can't ticket yet; graduates as the frontier advances -->

## Out of scope

<!-- see "Out of scope": work ruled beyond the destination; closed, never graduates -->
```

### Tickets

Each ticket is a file in the map's `tickets/` folder, named `NN-<slug>.md` and numbered from `01` in the order created; the number is its identity. Its body is the question, sized to one decision:

```markdown
# <ticket title>

Type: <research | prototype | grilling | task>
Status: open
Blocked by: <ticket numbers, comma-separated, or "none">

## Question

<the decision or investigation this ticket resolves>
```

The `Type:` line is one of `research`, `prototype`, `grilling`, `task` (see [Ticket Types](#ticket-types)).

The `Status:` line is one of `open`, `claimed`, `resolved`, or `out-of-scope`. A ticket is **closed** when it's `resolved` or `out-of-scope`.

A session **claims** a ticket by setting `Status: claimed` and saving the file, **first**, before any work, so concurrent sessions skip it. Re-read the file just before claiming it, in case another session got there first.

Blocking is the `Blocked by:` line. A ticket is **unblocked** when every ticket it lists is closed; the **frontier** is the tickets that are `open` and unblocked, the edge of the known.

The answer isn't part of the question; it's appended on resolution (see [Work through the map](#work-through-the-map)). Assets created while resolving a ticket are linked from the ticket file, not pasted in.

## Ticket Types

Every ticket is either **HITL** (human in the loop, worked _with_ a human who speaks for themselves) or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling agent that answers its own questions has broken this).

- **Research** (AFK): Reading documentation, third-party APIs, or local resources like knowledge bases to surface a fact a decision waits on. Resolved by a subagent that calls the Skill tool with "mp-good:research", given the absolute path to write its findings to: the map's `research/NN-<slug>.md`, using the ticket's own number and slug, instead of wherever the research skill would otherwise save them. Tell it to pass that path on to any agent it starts. The ticket's answer gives the conclusion and links the findings file (relative to the ticket: `../research/NN-<slug>.md`). Use when knowledge outside the current working directory is required.
- **Prototype** (HITL): Raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to (an outline, a rough take, a stub, or UI/logic code) by calling the Skill tool with "mp-good:prototype". Links the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Grilling** (HITL): Conversation. The default case. Always call the Skill tool twice, for "mp-good:grilling" and "mp-good:domain-modeling".
- **Task** (HITL or AFK): Manual work that must happen before a _decision_ can be made: nothing to decide, prototype, or research, but the discussion is blocked until it's done. Signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. This is the one type that _does_ rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). Resolved when the work is done; the answer records what was done and any resulting facts (credentials location, new URLs, row counts) later tickets depend on.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the **fog of war**: the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, one at a time, until the way to the destination is clear and no tickets remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It's the undiscovered frontier _toward_ the destination: everything here is in scope, just not sharp enough to ticket. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators reading where the effort is headed.

**Fog or ticket?** The test is whether you can state the question precisely now, _not_ whether you can answer it now.

- **Ticket when** the question is already sharp, even if it's blocked and you can't act on it yet.
- **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into ticket-sized pieces: it's coarser than a ticket, and one patch may graduate into several tickets, or none, once the frontier reaches it.

**Not yet specified** excludes what's already decided (Decisions so far), what's already a live ticket, and what's out of scope (the next section).

## Out of scope

Fog only ever gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope**: it isn't fog, and it doesn't belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work you've consciously ruled out of _this_ effort. Scope, not sharpness, lands it here.

Out-of-scope work never graduates (the frontier stops at the destination), so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.

Ruling something out of scope is a scoping act, not a step on the route. When a ticket that already exists turns out to sit past the destination (mis-scoped in while charting, or exposed by a resolution), **close it** by setting `Status: out-of-scope` (a closed ticket is unambiguously off the frontier) and leave one line in the **Out of scope** section: the gist plus why it's out of scope, linking the closed ticket. It stays out of **Decisions so far**, which records the route actually walked; a scope boundary isn't a step on it.

## Invocation

Two modes.

### Chart the map

User invokes with a loose idea.

1. **Name the destination.** Call the Skill tool twice, for "mp-good:grilling" and "mp-good:domain-modeling", to pin down what this map is finding its way to: the spec, decision, or change. The destination fixes the scope, so it's settled first.
2. **Map the frontier.** Grill again, **breadth-first** this time: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no fog** (the way to the destination is already clear, with no open decisions that hang on other decisions), you don't need a map. Stop and ask the user how they'd like to proceed.
3. **Create the map**: choose its folder (see [Where the map lives](#where-the-map-lives)) and write `map.md`, with Destination and Notes filled in, Decisions-so-far empty, the fog sketched into **Not yet specified**.
4. **Create the tickets you can specify now** as files in `tickets/`, each with its `Blocked by:` line. The `Blocked by:` lines sort them into the frontier and the blocked; everything you can't yet specify stays in the fog: the **Not yet specified** section.
5. **Fire the research subagents.** For each `research` ticket you just created, spin up a subagent with a fresh context window that calls the Skill tool with "mp-good:research" to resolve it in parallel. Give each subagent the path to its ticket file and tell it to claim the ticket first (`Status: claimed`), then append its answer under `## Answer` and set `Status: resolved`, as in [Work through the map](#work-through-the-map); subagents don't edit `map.md`. Also give it the findings path described under [Ticket Types](#ticket-types).
6. As research subagents report back, append each one's context pointer to the map's Decisions-so-far yourself.
7. Print the full path to `map.md`, then offer to start working through the map, beginning with the first frontier ticket.

### Work through the map

User invokes with a map (the path to its folder or its `map.md`). A ticket is **optional**: without one, you pick the next decision, not the user.

1. Load the **map**: the low-res view, not every ticket body.
2. Choose the ticket. If the user named one, use it. Otherwise take the first frontier ticket in order. **Claim it**: set `Status: claimed` before any work.
3. Resolve it as the type its `Type:` line names (see [Ticket Types](#ticket-types)). **Zoom as needed**: read the full file of any related or closed ticket on demand; call the Skill tool for whichever skills the `## Notes` block names. If in doubt, call the Skill tool twice, for "mp-good:grilling" and "mp-good:domain-modeling".
4. Record the resolution: append the answer to the ticket under an `## Answer` heading, set `Status: resolved`, and **append a context pointer** to the map's Decisions-so-far.
5. Add newly-surfaced tickets, numbered after the highest existing one; graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its new ticket. If the answer reveals that a ticket (this one or another) sits beyond the destination, **rule it out of scope** rather than resolving it on the route. If the decision invalidates other parts of the map, update or delete those tickets.
6. Go back to step 2 and take the first frontier ticket in order (a ticket the user named applies only to the first pass). Stop when the user says to, or when the frontier is empty. If it's empty, tell the user why: the map is complete, or the remaining tickets are blocked, claimed by another session, or still fog.

The user may run unblocked tickets in parallel, so expect other sessions to be editing the map's files concurrently.

# mp-good

A Claude Code plugin with the parts of [Matt Pocock's skills](https://github.com/mattpocock/skills) I actually use, adapted to my preferences and made to work without the rest of his workspace.

## Skills

User-invoked:

- `/mp-good:grill-me`: a relentless interview to sharpen a plan or design.
- `/mp-good:improve-codebase-architecture`: scan a codebase for deepening opportunities, present them as an HTML report, then grill through the one you pick.
- `/mp-good:wayfinder`: chart a large, foggy effort as a map of decision tickets, then resolve them one at a time.

Called by the skills above (and usable on their own): `grilling`, `codebase-design`, `domain-modeling`, `research`, `prototype`.

## Install

```
/plugin marketplace add <path-or-url-to-this-repo>
/plugin install mp-good@mp-good
```

## Relationship to upstream

These skills started as copies of Matt Pocock's, under his MIT license (see [LICENSE](LICENSE)). [UPSTREAM.md](UPSTREAM.md) records which upstream commit they came from, where each file came from, every change made here and why, and how to bring in later upstream changes.

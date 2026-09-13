# Context pack format

_Portable core. Version 1, 2026-09-12. One pack per client or project, at `context/<slug>.md`. The pack is the agent's **primary source** for that project and its **cross-client boundary**: briefs draw their inputs from it, executing agents read what it names, and nothing from another pack ever reaches this project's briefs, comments, or artifacts._

## Which pack applies

An issue carries the label `project:<slug>`; the pack is `context/<slug>.md`. Sub-issues inherit the label from their parent at creation. An issue with no project label cannot be briefed: the pass asks which project it belongs to (a project-level needs-info) unless the operator named a default pack for the pass. In Buganizer the label's equivalent is the component or a hotlist; see `port-log.md`.

## Operator level

`context/operator.md` declares what applies to every project: the operator's standing skills and rule-sets (writing style, house rules). Every brief inherits them; a project pack adds its own; a brief names only task-specific additions.

## Sections (all present; write "none" rather than omitting)

```markdown
# Context pack: <Project name>

**Slug:** <slug>            (matches the label `project:<slug>` and this file's name)
**Client:** <client name, or "internal">
**Operator:** <who runs this project>
**Deliverables folder:** <where artifacts go; the brief's Output names it>
**Delegated document types:** <document types the agent may draft for this client even though they are client-facing, or "none">

## Glossary
| Term | Meaning here |
Terms the project uses. Briefs use these words; the isolation check treats them as this project's.

## Stakeholders
| Name | Role | Notes |
People and their roles. Names are project-private.

## Sources
| Name | What it is | Where |
Named sources the agent may read: Drive folders, documents, wikis, the tracker. "Where" is the location a brief's Inputs cite.

## Standing skills and rules
- `rules/<file>.md`: project-level rule-sets, in addition to the operator's.

## Out of bounds
- What the agent must not read or draw on for this project. Other clients are always out of bounds; list anything else.
```

## How the pack is used

| Stage | Use |
|---|---|
| Triage and decomposition | Read the pack before prior-art search. Inputs in a brief come from Sources (by name or location) or from a link the issue itself carries. If the pack lacks what a brief needs, the brief's Rationale says "not in the context pack" and the item is needs-info, never a guess. |
| Approval | Each leaf's brief is checked against the parent's pack the same way. |
| Execution | The agent reads only the brief's Inputs. Deliverables are checked: no other pack's sources, stakeholders, or terms may appear. |
| Rules | Every brief's "Inherited" line names the operator's rule-sets and the pack's; `apply` refuses a brief that omits one. Executing agents load them all; hard rules bind. |

## Isolation check (mechanical)

For a project A issue, the check refuses any brief, proposal, question comment, or delivered artifact that contains, case-insensitively, another pack's: source name or location, stakeholder name, glossary term, client name, or slug, unless project A's own pack also lists the same string. It is a guard against leakage, not a substitute for the agent reading only what the brief names.

## Open at port time (deliberately not decided)

How strictly the pack bounds reads **beyond** its listed sources. v1 enforces: inputs come from the pack or the issue's own links, and nothing from another pack leaks out. It does not stop an agent from opening an unlisted document inside its own client's Drive. Whether to tighten that to "listed sources only" is teased out on the corporate side, where the read surface is different; do not over-restrict before then.

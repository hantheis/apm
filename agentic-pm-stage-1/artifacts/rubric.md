# Triage rubric

**Version:** 2
**Last amended:** 2026-09-13 (amendment from `LAS-14`)
**Amendment log:** see the end of this file.

_Portable core. Read in full on every pass. The human edits this file directly; the agent proposes amendments (as `#### Rubric amendment` comments on the issue where it was overruled, see `rubric-amendment-template.md`) and applies them only after the human replies "approve", bumping the version each time._

## Outcomes

Every item is classified into exactly one of:

- **agent-capable** (`apm:agent`): all five criteria below hold. Post a brief.
- **human-only** (`apm:human`): the item is clear and briefable but fails criterion 3, 4, or 5, or turns on judgement only a human should exercise. Post a context brief.
- **needs-info** (`apm:needs-info`): the item cannot be briefed because criterion 1 or 2 fails for want of information. Post specific questions and wait.
- **too big** (`apm:too-big`): the item cannot be judged as a whole. Decompose it and classify the leaves.

Decide in this order: is it too big? If not, can it be briefed at all (needs-info)? If so, do all five criteria hold (agent) or not (human)?

## Agent-capable criteria

An item is agent-capable only when **all** of these hold. Cite the failing criterion by number in the rationale when one does not.

1. **Verifiable from the brief alone.** Success can be stated as acceptance criteria a reviewer can tick without asking the requester what they meant.
2. **Inputs are reachable.** Every document, prior ticket, or source the work needs is linked, named in the context pack, or findable in prior tickets within the pass budget. No locked-up context in someone's head.
3. **Output is reviewable text or documents.** A PRD, brief, status update, summary, research findings, or a comms draft. Not code. A decision is agent-capable as a recommendation: the output is a written recommendation with its reasons, drawn from the named inputs, for the human to accept or change; the decision itself stays the human's.
4. **No irreversible external action.** Nothing is sent, published, deleted, or committed outside the tracker and the artifact store. Comms are drafts.
5. **Fits the effort budget.** Completable within the caps below in one sitting. If it needs more, it is too big, not a bigger budget.

## Scope guard (v1)

Regardless of the criteria, these are never agent-capable in v1:

- Code tickets.
- Research that needs external web sources. Internal sources only (Drive, prior tickets, intranet).
- Anything whose acceptance criteria require sending, publishing, or deleting.

## Budgets

Objective caps. "One sitting" is the intuition; the numbers are the rule. Tunable: change them here and note it in the amendment log.

| Field | Default | Applies to |
|---|---|---|
| `turn_cap` | 40 agent turns | one execution of one brief |
| `tool_call_cap` | 80 tool calls | one execution of one brief |
| `decomposition_depth_cap` | 3 levels below the root issue | one decomposition |
| `decomposition_loop_cap` | 5 proposal revisions on one parent | one approval loop |
| `pass_issue_cap` | 25 issues read per pass | one triage pass |
| `prior_art_cap` | 15 prior tickets opened per item | the prior-art search for one item |

On any cap: stop, post where it got stuck as a comment, and either re-decompose (with approval) or hand to a human. Never push through.

## Prior-art line

Before proposing anything for an item: read the project's context pack first (primary source), then search prior tickets in the tracker up to `prior_art_cap`, and link related or duplicate issues in the brief. Never search indefinitely.

## Amendment log

| Version | Date | Change | Origin |
|---|---|---|---|
| 1 | 2026-09-12 | Initial criteria, scope guard, and budget defaults from the design summary. | build |
| 2 | 2026-09-13 | Not code. A decision is agent-capable as a recommendation: the output is a writt | overrule on `LAS-14` |

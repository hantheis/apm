# Proposal template

_Portable core. Version 2, 2026-09-12 (v1 carried a full brief per leaf; Hannah asked for a lighter, bulleted proposal so effort is not spent on briefs until the split is agreed). The decomposition of a too-big issue, posted as one comment on the parent. The human steers or approves in reply. Sub-issues, and their full briefs, are created only after approval._

Each leaf is a **sketch**: enough to judge the split, not a brief. Full briefs are written at approval (ticket 06), one per sub-issue, from the brief template. A leaf that is still too big is not a leaf: split it further, or if the depth cap is reached, post the stuck template instead.

```markdown
#### Proposal

**Rubric version:** 1
**Depth:** 1 of 3
**Related prior tickets**
- `NN` (title): how it relates (done and reusable, duplicate, shares a source). Or "none found within budget".

##### Leaf 1: Short title
- **Classification:** agent-capable | human-only
- **Goal:** one line, what is made and for whom.
- **Done when:** one line, the main acceptance criterion.
- **Depends on:** Leaf N, Leaf M | none
- **Why:** one line: which criteria hold, or for human-only, which one fails and why.

##### Leaf 2: Short title
(same five bullets)

**Dependencies between leaves**
- Leaf 2 depends on Leaf 1: why.

**To approve:** start your reply with "approve"; add "Leaf 3 is mine" or "Leaf 2: agent" to change who does a leaf. **To steer:** reply with what to change in plain words and the next pass posts a revised proposal. (Full rules: `replying.md`.)
```

## Notes for the writer

- Five bullets per leaf, no more. If a leaf needs a paragraph to explain, it is probably two leaves or needs-info.
- Assignee kind is not stated: agent-capable means agent, human-only means human. The human changes it at approval if they want ("Leaf 3 is mine"), and the full brief records the confirmed kind.
- Leaves are numbered flat, in a sensible working order. Nesting is expressed in `Depends on`, not in headings.
- `**Depth:**` is the deepest level of splitting needed to reach these leaves, against the rubric's `decomposition_depth_cap`.
- Prior art at the top is for the parent as a whole. Where a leaf reuses a prior ticket, say so in its `Why`.
- At approval, each leaf's full brief inherits these lines: Goal expands the one-liner, Acceptance criteria start from Done when, Dependencies come from Depends on, Rationale expands Why.

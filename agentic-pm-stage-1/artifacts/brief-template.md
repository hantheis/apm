# Brief template

_Portable core. Version 1, 2026-09-12. One format for agent briefs and human context briefs. Once approved, the brief is the contract and the original issue text is context._

A brief is posted as a comment on the issue at triage (or inside the proposal comment during decomposition) and copied into a sub-issue body at creation. It must carry every section below, in this order, with these exact headings, so a reader and a checker can find each part. Write "none" rather than omitting a section.

```markdown
#### Brief

**Classification:** agent-capable | human-only
**Assignee kind:** agent | human
**Rubric version:** 1

**Goal**
One paragraph. What is being made and for whom. Not how.

**Acceptance criteria**
- [ ] Each criterion is tickable by a reviewer from the output alone.
- [ ] At least one criterion names the output form (Doc, comment).

**Inputs**
- Links or named sources drawn from the context pack, one per line. Say what each is for.

**Related prior tickets**
- `NN`: why it is related (duplicate, predecessor, shares a source). Or "none found within budget".

**Skills and rules**
- Inherited: the standing skills and rule-sets the context pack declares (by name; do not restate them).
- Task-specific: anything extra this task needs. Or "none".

**Output**
Form and destination. For example: "Google Doc in the project's deliverables folder, linked from this issue with a three-line summary comment" or "an issue comment".

**Budget**
turn_cap N, tool_call_cap N (from the rubric unless the brief narrows them).

**Dependencies**
- Blocked by `NN` (title): why. Or "none". This text copy is authoritative; the tracker relation is convenience.

**Rationale**
Why this classification: name each rubric criterion and how it holds or fails. Two to five lines.
```

## Notes for the writer

- The brief is written from the rubric and the context pack, never from assumptions. If an input cannot be named, the item is needs-info, not a brief with a gap.
- A human context brief uses the same template. The Goal and Inputs sections do the briefing; the Rationale says which criterion made it human-only.
- Keep the classification and assignee kind separate. The classification is the agent's recommendation; the assignee kind is confirmed by the human at approval.

# Label vocabulary

_Portable core. Version 1, 2026-09-12. Settled at eight labels during spec review._

Workflow state lives in labels, not tracker statuses. Every label is prefixed `apm:` so it is recognisable in any tracker (labels in Linear, hotlists in Buganizer). The labels must exist in the tracker before a pass runs; the boundary applies and removes them, it never creates them.

A label answers one of three questions. An issue may carry one label from each group at a time.

## What the agent thinks of it (classification)

| Label | Meaning | Set by | Cleared when |
|---|---|---|---|
| `apm:agent` | Agent-capable. Passes every rubric criterion. A brief has been posted. | agent at triage; human may overrule | the issue is done |
| `apm:human` | Human-only. Fails a rubric criterion on judgement or irreversible action. A context brief has been posted. | agent at triage; human may overrule | the issue is done |
| `apm:needs-info` | The agent cannot brief it. Its specific questions are posted as a comment and it waits. Nothing is decomposed on guesswork. | agent at triage | the human answers and the next pass reclassifies |
| `apm:too-big` | Cannot be classified as a whole. With `apm:proposed`: a decomposition awaits the human. Without it: the agent hit its depth cap, posted a stuck comment, and handed the split to the human. | agent at triage | the decomposition is approved and sub-issues exist |

## Where it is in the approval loop (parents only)

| Label | Meaning | Set by | Cleared when |
|---|---|---|---|
| `apm:proposed` | A decomposition proposal is the latest agent comment on this parent and awaits the human's steer or approval. | agent | replaced by `apm:approved` |
| `apm:approved` | The human approved the split. Sub-issues have been created, the master checklist is written on the parent. | agent, on seeing the approval comment | the parent is done, or the human says "re-decompose" (back to `apm:proposed`) |

## Where it is in execution (leaves only)

| Label | Meaning | Set by | Cleared when |
|---|---|---|---|
| `apm:blocked` | Waiting on another task named in its blocked-by relation and its brief. The parent carries a loud-blocking comment. | agent at sub-issue creation or on a later pass | the blocker is done |
| `apm:review` | An agent finished within budget and handed the output back. The human gives feedback (retry) or flips status to done. | executing agent | the human sets status to done, or feedback triggers a retry (label removed, re-added when the retry finishes) |

## Project labels (not workflow state)

`project:<slug>` names the context pack an issue belongs to (`context/<slug>.md`). Set by whoever files the issue; sub-issues inherit it at creation. One per issue. In Buganizer the equivalent is the component or a hotlist. Not part of the eight-label workflow vocabulary; listed here because the boundary applies it.

## Rules

- **Overruling.** The human may change any classification label directly. The next pass treats a changed label as an overrule and proposes a rubric amendment (ticket 09).
- **Done is not a label.** Done is the tracker's own status, set by the human, never by the agent.
- **No other `apm:` labels.** If the protocol needs a new state, it is added here first, with its group, setter, and clearing condition.

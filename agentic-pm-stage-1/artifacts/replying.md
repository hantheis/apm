# Replying to the agent

_Portable core. Version 2, 13 September 2026. How the human replies in issue comments so the next pass does the right thing. Four keywords (approve, re-decompose, reject, file), everything else is plain prose._

The agent reads the **latest human comment** after its own last comment on an issue. Where it appears decides what happens.

## On a proposal (`apm:proposed`)

| You want | Write | What the next pass does |
|---|---|---|
| Approve the split as proposed | A comment **starting** with `approve` or `approved` | Writes full briefs, creates the sub-issues, posts the master checklist and any loud-blocking comment, moves the parent to `apm:approved`. |
| Approve but change who does a leaf | Same, plus `Leaf 3 is mine`, `Leaf 3: human`, or `Leaf 2: agent` anywhere in the comment | Your call wins over the agent's classification and is marked "(your call)" in the checklist. |
| Change the split | Anything else. Plain prose: "merge leaves 1 and 2", "leaf 4 before 3", "drop the research leaf" | Posts one revised proposal. The parent stays `apm:proposed`. Up to five proposals; then the agent posts a stuck comment and hands over. |

**Pitfall:** `approve` must start a line. "I think we can approve this" is read as a steer and you get another proposal.

## On an approved parent (`apm:approved`)

| You want | Write | What the next pass does |
|---|---|---|
| Redo the split | A comment **starting** with `re-decompose` (or `redecompose`), then what changed | Posts a fresh proposal saying which existing sub-issues are kept, moves the parent back to `apm:proposed`. Existing sub-issues stay for you to close or cancel. |
| Anything else | Any other comment | Nothing happens to the split. Feedback on a task belongs on that task's sub-issue. |

## On a needs-info issue (`apm:needs-info`)

No keyword. Any reply reopens it; the agent reads your answers and classifies afresh, swapping the label.

## On a stuck issue (`apm:too-big` without `apm:proposed`)

No keyword. Reply with the narrowing or the missing names the stuck comment asked for, and the next pass decomposes again from the top.

## On a task in review (`apm:review`)

| You want | Do | What the next pass does |
|---|---|---|
| Changes | Comment with the feedback, in plain words | The next execute pass retries that task and delivers a new version, saying which feedback it addressed. |
| Changes you left in the Doc | Comment "the comments are in the Doc" | The retry reads the Doc's inline comments instead. |
| File the next steps it proposed | Comment **starting** `file them`, or `file 1, 3` for some | Creates one issue per item under the same project, untriaged, and posts `#### Filed` with the ids. The next triage pass classifies and briefs them; the delivery's agent/human tag was only a suggestion. Not read as feedback. |
| Accept it | Flip the status to done (or fixed, closed, resolved) | The parent's checklist is ticked and the task's final comment becomes its outcome record. Nothing else needed. |
| Reject it | Flip the status to a cancel-type status | Same, with `result=rejected` in the record. |

Nothing you write on a sub-issue changes the parent's split.

## On a human task of yours (`apm:human`)

Do the work, then flip the status to done. If other tasks wait on yours, the agent posts one nudge saying which; replying to it resets the nudge, and flipping it to done unblocks them on the next pass.

## Overruling a classification

Change the label yourself (`apm:agent` to `apm:human`, or back). No comment needed. The next pass posts one `#### Rubric amendment` on that issue: the criterion it thinks you applied and the wording change that would make the rubric agree with you next time.

| You want | Write | What the next pass does |
|---|---|---|
| Make it the rule | A comment **starting** `approve` | Edits `rubric.md`, bumps its version, logs the change, and says which version it became. |
| Keep it a one-off | A comment **starting** `reject` | Records the rejection. The rubric is unchanged; your label stands. |
| Think about it | Anything else, or nothing | Waits. Your label already stands either way. |

## Bigger steers

A chat session referencing the issue number is fine for anything that does not fit a comment box. The agent writes the resolved outcome back to the issue as a comment so the tracker never falls behind the conversation.

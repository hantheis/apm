# Workflow protocol

_Portable core. Version 1, 2026-09-12. This text is the protocol. Shells (Claude Code skills in v1, Antigravity workflows later) invoke it; they add no logic of their own. Mechanical steps marked (M) may be automated per shell; judgement steps marked (J) are the agent's, made from the rubric and the context pack._

## Inputs to every pass

- The tracker, through the tracker contract (`tracker-contract.md`); for execution, also the artifact store (`artifact-store-contract.md`).
- The rubric (`rubric.md`), read in full.
- The label vocabulary (`labels.md`).
- The brief template (`brief-template.md`).
- The project's context pack (`context/<slug>.md`, named by the issue's `project:<slug>` label; format in `context-pack-format.md`) and the operator's `context/operator.md`: the primary source and the read boundary for that project. Every brief, comment, and artifact is checked against the cross-client boundary before it is posted.
- The agent's author name in the tracker (`apm`), so it can recognise its own prior comments.

## The triage pass

Invoked manually in stage 1 ("run a triage pass"). In stage 2 the same steps run per issue from a webhook handler. Nothing below depends on how it was invoked.

1. **Read the rubric and the context pack.** (M) Note the rubric version; every brief records it.
2. **List candidates.** (M) Candidate = an issue that is not done and needs an action from the agent: an untriaged issue (triage), a needs-info issue the human has answered (reclassify), a proposed issue the human has replied to (steer or approve), or an approved issue the human has told to re-decompose. Sub-issues of an unapproved parent are never candidates. Read at most `pass_issue_cap` candidates, lowest id first.
3. **Per candidate, read your own prior state first.** (M) Read the issue with its comments and labels. If the agent already posted a brief, questions, or a proposal on it, and the human has not commented since, skip it. This is the idempotency rule: a re-run or an overlapping pass adds nothing to an issue already handled. A `apm:needs-info` issue is the one classified state that reopens: a human comment after the questions makes it a candidate again, and the next classification replaces `apm:needs-info` with the new label. (Other classified issues stay handled; a changed label is the overrule flow, ticket 09.)
4. **Prior art.** (J, bounded) Read the context pack sources named for this item's project. Then search prior tickets up to `prior_art_cap`, and note related or duplicate issues for the brief.
5. **Classify.** (J) Apply the rubric in its order: too big, needs-info, then agent versus human. Write the rationale citing criteria by number.
6. **Act.** (M after J) Exactly one of:
   - **agent-capable**: post one brief (template, classification agent-capable, assignee kind agent); add `apm:agent`.
   - **human-only**: post one context brief (template, classification human-only, assignee kind human, Rationale naming the failing criterion by number); add `apm:human`.
   - **needs-info**: post one comment headed `#### Questions` naming the failing criteria and listing the specific questions, nothing else; add `apm:needs-info`. No brief, no decomposition, no other label.
   - **too big**: decompose (below), post one proposal comment; add `apm:too-big` and `apm:proposed`.
   Never post a second brief, question set, or proposal on an issue that already has one without a human comment in between.
7. **Digest.** (M) End the pass with one summary, printed to the operator (and, in stage 2, posted as the daily digest):
   - Triaged: N, listing id and label.
   - Proposed: M, listing parent ids awaiting approval.
   - In review: K, listing sub-issues delivered and awaiting the human.
   - Ready for agent: approved, unblocked `apm:agent` sub-issues not yet executed.
   - Complete: approved parents whose sub-issues are all done, for the human to flip.
   - Blocked on you: issues waiting on a human comment (proposals, needs-info, stuck), and human-assigned tasks that gate other work.
   - Aging needs-info: `apm:needs-info` issues with days since the questions were posted.

## Decomposition

Split a too-big item into leaves until every leaf is confidently agent-capable or human-only. (J) Work top down: split the item, classify each part, split any part still too big, and count each split as one level of depth. Stop at `decomposition_depth_cap`.

- **Prior art first.** Before writing the proposal, read the context pack's named sources and search prior tickets up to `prior_art_cap`. A leaf that duplicates a done or open ticket is linked, not re-proposed.
- **The proposal** is one comment on the parent, headed `#### Proposal` (`proposal-template.md`): rubric version, depth reached, related prior tickets, then one `##### Leaf N: title` sketch per leaf, as five bullets (classification, one-line goal, one-line done-when, depends-on, one-line why), then dependencies between leaves in text. Add `apm:too-big` and `apm:proposed`. **No sub-issues and no full briefs exist yet**: the proposal is for judging the split, and effort on briefs is spent only once the split is agreed.
- **Stuck.** If any branch is still too big at the depth cap, do not post a partial tree. Post one comment headed `#### Stuck` (`stuck-template.md`) naming the depth reached, the branch that would not resolve, what was tried, and `**Handing to:** human`. Add `apm:too-big` only. The human splits it or narrows the issue; their comment reopens it.
- **Idempotent.** A proposed or stuck issue is skipped on re-run until the human comments.

## Approval loop

The human replies to the proposal on the parent. What they say decides the next pass (the human-facing version of these rules is `replying.md`):

- **Steer**: any reply that is not an approval. (J) Revise the proposal in the light of it and post one revised proposal comment (same template); the parent stays `apm:proposed`. At most `decomposition_loop_cap` proposals per parent; after that, post a stuck comment and hand over.
- **Approve**: a reply beginning "approve" (or "approved"). It may override any leaf's assignee kind: "Leaf 3 is mine", "Leaf 2: agent". The override wins over the agent's classification and is marked "(your call)" in the checklist. Then, in this order:
  1. (J) Write one full brief per leaf from the brief template, expanding the sketch: Goal from the one-liner, Acceptance criteria starting from Done when, Dependencies naming the leaves from Depends on ("Blocked by Leaf 1: why"), Rationale from Why. A human-only leaf gets a context brief with the failing criterion named.
  2. (M) Every brief is checked against the template and the (possibly overridden) sketch before anything is created. One bad brief means nothing is created.
  3. (M) Create one sub-issue per leaf under the parent: title from the leaf, body = "Leaf N of `parent`" plus the brief, label `apm:agent` or `apm:human`. Record each dependency as a native blocked-by relation (convenience) while the brief text (authoritative) already names it. Label every leaf with an unmet dependency `apm:blocked`.
  4. (M) Post the **master checklist** on the parent, headed `#### Master checklist`: one unchecked item per leaf with its sub-issue id and kind. This is the shared picture of the work; it is ticked as sub-issues are closed (review stage).
  5. (M) If any leaf is blocked, post one **loud-blocking** comment on the parent, headed `#### Blocked`: "N tasks waiting:" then one line per blocked sub-issue naming what it waits on. Never a silent hold.
  6. (M) Swap `apm:proposed` for `apm:approved`.
- **After approval** the parent is handled. Further comments do nothing to the split; feedback belongs on the sub-issues. The one exception is a reply beginning "re-decompose": the next pass posts a fresh proposal and swaps `apm:approved` back to `apm:proposed`. Existing sub-issues are left for the human to close or cancel; the new proposal should say which are kept.

## Execution

Run one `apm:agent` sub-issue **from its brief alone**. The brief is the contract; the parent and the proposal are not consulted.

1. (M) Load the brief. Refuse if the issue is not `apm:agent`, is `apm:blocked`, or names a rule-set that cannot be found. Load every named rule-set in full: **hard rules are binding**, on par with the rubric.
2. (J) Read only the inputs the brief names. Produce the output in the form and type the brief's Output names, from the template that resolves for that type (`templates/README.md`: project override, then operator override, then core). A **document** goes to the artifact store in the folder the brief names; a **comment** is posted on the issue. **Sized to fit:** when the Output says so (research, typically), the result is a comment if it is under the template's comment threshold in words and an artifact otherwise, with the same section headings either way. Comms are drafts: the template forbids any send instruction.
3. (M) Budget. The caps are objective (`turn_cap`, `tool_call_cap` from the brief). In v1 the agent counts and reports them; delivery over either cap is refused and must go through the stuck path. On a budget hit: stop, post the stuck comment (`stuck-template.md`, execution form: budget consumed, stopped at, what was tried, remaining, handing to human). No partial result is ever delivered as done.
4. (J) **Self-check.** Before delivering, tick every acceptance criterion in the brief and every hard rule in the loaded rule-sets against the output, one line each with a one-line reason (`#### Self-check`, sections **Acceptance criteria** and **Hard rules**; a rule line reads `<rule-set path> hard rule N: reason`). An unticked line means the output is not done: iterate and re-check inside the caps; a miss that cannot be fixed inside the caps goes through the stuck path. The self-check is the agent's own gate, so the reviewer starts from what was verified, not from scratch.
5. (M) Deliver: the self-check is checked first and the delivery refused if it is missing, does not cover every acceptance criterion and hard rule, lacks a reason on any line, or carries an unticked line; then the artifact is checked against its template (required sections in order, forbidden phrases) and refused if it fails; then a document delivery posts `#### Delivered` with a **three-line summary** (what it is, what it says, what to check), the artifact link, the budget consumed, and the rule-sets applied; a comment delivery posts the output itself with the same trailer. The ticked self-check follows the trailer in both. (J) The delivery may end with `#### Next steps`: numbered items, each `<title> [agent|human]: <why>`, for follow-on work the output points at, pre-classified as a recommendation; (M) a malformed block is refused and the block passes the isolation guard. Nothing is filed until the human says so (review stage). Then add `apm:review`; `apm:agent` stays. The agent never sets a status.
6. Idempotent: a delivered issue is not delivered again; an unanswered stuck comment is not repeated.

## Review

The human closes the loop on each sub-issue. Every pass runs the review mechanics first (M), before triage, so the tracker reflects what the human did since the last pass.

- **File next steps.** A human comment starting `file` after a delivery (`file them`, `file 1, 3`) is not feedback. The next pass creates one root issue per chosen next step through `create_issue`, under the same project label, with an intake body naming the delivery it follows on from and the suggested kind, untriaged; then posts `#### Filed` with the new ids (or "nothing to file"). The next triage pass classifies and briefs them like any other issue: the delivery's kind is a recommendation, the rubric decides. Idempotent: the `#### Filed` comment answers the reply.
- **Feedback** is any other human comment on an `apm:review` sub-issue after the latest delivery. The next execute pass **retries that task**: it reloads the brief, reads the feedback, produces a new version (the store never overwrites), and delivers again with `**Retry:** N` and the first line of the feedback in the delivery comment. Retries are counted from the delivery comments. Feedback never re-decomposes.
- **"The comments are in the Doc."** A feedback comment saying so (any phrasing containing "comments are in the Doc") redirects the retry to the artifact's inline comments, read through the artifact store (`comments`). If the artifact has none, the retry is refused and says so.
- **Done** is the human flipping the sub-issue's status to one of the tracker's done statuses. On the next pass: (1) the sub-issue's final comment becomes the **outcome record**, one line: `Outcome: retries=N budget=<consumed> steered=yes|no result=approved|rejected` (rejected when the done status is a cancel-type status); (2) the parent's **master checklist is re-posted with the item ticked** (comments are append-only, so the latest checklist comment is the current one; it is re-posted only when something changed); (3) sub-issues whose blockers are all done lose `apm:blocked`, and the parent's loud-blocking comment is re-posted to say what still waits, or that nothing does.
- **Complete.** When every sub-issue is done the checklist is fully ticked and the digest lists the parent under "Complete" for the human to flip. The agent never sets a status.
- **Nudges.** An open `apm:human` sub-issue that other sub-issues wait on gets one `#### Nudge` comment naming what it gates. It is not repeated until the human replies on it. The digest's blocked-on-you list names it as "your task, gating `NN`".
- **Idempotent.** A pass never re-appends an outcome record, re-ticks a checklist that has not changed, or re-nudges without a human reply in between.

## Rubric feedback

The rubric learns from being overruled. The human overrules by changing the agent's classification label (`apm:agent` to `apm:human`, or back); nothing else is needed.

1. (M) On the next pass an issue whose label disagrees with the classification in the agent's latest brief is an **overrule**. It is a candidate with action "overruled".
2. (J) The agent works out which rubric criterion (1 to 5, the scope guard, or the budgets) would have produced the human's answer, and what wording change would make it do so next time. It posts one `#### Rubric amendment` (`rubric-amendment-template.md`): the overrule, the criterion, the current wording quoted exactly, the proposed wording, and why. Then it waits. Never twice on one issue.
3. The human replies **approve** or **reject** at the start of a line. (M) Approve: the current wording is replaced in `rubric.md` (it must appear there exactly once), the version is incremented, the amendment log gains a row naming the issue, and the issue gets a comment saying which version it became. Reject: a comment records that the rubric is unchanged and the overrule stands as a one-off. Any other reply keeps waiting.
4. The overruled issue's eventual outcome record says `steered=yes`.
5. The human may also edit the rubric directly at any time; the version line is theirs to bump.

## Invariants (what tests pin down)

- Sub-issues never exist before approval.
- A re-run pass adds nothing to an issue it already handled.
- A budget hit produces a stuck comment and a handoff, never a completed task.
- A needs-info issue carries questions and nothing else.
- A closed task's final comment carries the outcome record.
- Feedback produces a retry and a new artifact version, never a re-decomposition.
- The rubric changes only on an approved amendment, and then its version increments.
- A blocked parent carries a loud-blocking comment.
- A brief for project A names no source from project B's context pack.

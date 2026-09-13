---
name: triage-queue
description: Run one agentic-PM triage pass over a local-markdown issue directory. Use when the operator says "run a triage pass", "triage the queue", or "/triage-queue <issues_dir> [context_pack]".
---

# /triage-queue

Thin Claude Code shell over `artifacts/workflow-protocol.md`. You add no logic of your own: the protocol says what to do, the rubric says how to judge, the template says what to write. All tracker writes go through the boundary commands below; never edit issue files by hand.

Arguments: `<issues_dir>` (required), `[default_pack]` (a slug, used only for issues with no `project:` label; omit to have those asked which project they belong to).

The context pack for an issue is `artifacts/context/<slug>.md` where the issue carries the label `project:<slug>` (`plan` prints it per candidate). `artifacts/context/operator.md` applies to every project. Read `artifacts/context-pack-format.md` once.

On Linear (ticket 13 onward) the same steps apply, but you perform the boundary operations yourself through the connector following `artifacts/trackers/linear.md`, and you sign every comment with a final line `-- apm`.

## Steps

1. Read in full: `artifacts/rubric.md` (note its version), `artifacts/labels.md`, `artifacts/brief-template.md`, `artifacts/context/operator.md`, and each candidate's context pack. The pack is your primary source and your read boundary for that project: cite only its sources (or links the issue itself carries) as Inputs, name its rule-sets and the operator's under Inherited, and never mention another client's sources, people, or terms. An issue with no project label and no default pack gets a needs-info comment asking which project it belongs to.
2. Run `python3 -m apm.review pass <issues_dir>` first: it appends outcome records to done sub-issues, ticks parent checklists, unblocks dependents, and nudges gating human tasks. Report what it did. Then run `python3 -m apm.triage plan <issues_dir>`. Keep the `started:` timestamp. The list is every issue that still needs a decision; issues you already handled are excluded (idempotency). Each line may carry an action: none means triage; `[reclassify]`, `[steer]`, `[approve]`, `[redecompose]`, `[overruled]`, `[amend]` mean the human did something and you act on it (step 3b).
3. For each candidate, lowest id first:
   1. Read it: `python3 -c "from apm.tracker import LocalMarkdownTracker as T; i=T('<issues_dir>').read_issue('<id>'); print(i.title); print(i.body); print(i.labels); [print(c.author, c.timestamp, c.body) for c in i.comments]"`.
   2. Prior art, bounded by `prior_art_cap`: read the context pack sources this item names, then skim other issues in the directory for related or duplicate work.
   3. Classify in the rubric's order: too big, needs-info, agent versus human.
   4. Write the comment to a scratch file:
      - agent or human: the brief, from `artifacts/brief-template.md`, every section present in order, starting `#### Brief`.
      - needs-info: `#### Questions`, the failing criteria by number, then a numbered list of the specific questions (see `artifacts/needs-info-example.md`).
      - human-only: the brief with Rationale naming the failing criterion by number (see `artifacts/brief-example-human.md`). A human comment answering questions on a needs-info issue puts it back in the plan; classify it afresh and `apply` swaps the label.
      - too-big: decompose top down, at most `decomposition_depth_cap` levels, prior art first. Write the proposal from `artifacts/proposal-template.md` (see `artifacts/proposal-example.md`): a five-bullet sketch per leaf, classified agent-capable or human-only, no assignee kind. No full briefs; those come at approval. Outcome `too-big`. Never create sub-issues here.
      - stuck: if a branch is still too big at the depth cap, write `artifacts/stuck-template.md` instead (see `artifacts/stuck-example.md`) and use outcome `stuck`. No partial tree.
   5. Apply: `python3 -m apm.triage apply <issues_dir> <id> <agent|human|needs-info|too-big|stuck> <scratch_file> [artifacts/context] [default_pack]`. If it prints "already handled", move on. If it prints "rejected" (format, a missing inherited rule-set, an input not in the pack, or a cross-client mention), fix the comment; do not bypass the check. If the pack lacks an input you need, say "not in the context pack" in the Rationale and classify needs-info.
3b. For a candidate marked `[steer]`, `[approve]`, or `[redecompose]`:
   - `[steer]`: read the human's reply, revise the proposal, and `apply` it with outcome `too-big` (one revised proposal comment). If `apply` says the loop cap is reached, post a stuck comment instead.
   - `[approve]`: run `python3 -m apm.approval leaves <issues_dir> <id>` to see the leaves with the human's overrides applied. Write one full brief per leaf to `<scratch>/leaf-N.md` from `artifacts/brief-template.md`, expanding the sketch (Dependencies must name the leaves from the sketch's Depends on; a human-only leaf names the failing criterion). Then `python3 -m apm.approval approve <issues_dir> <id> <scratch>`. It creates the sub-issues, the master checklist, and the loud-blocking comment, or rejects everything if one brief is wrong.
   - `[redecompose]`: write a fresh proposal saying which existing sub-issues are kept, and `apply` with outcome `too-big`.
   - `[overruled]`: the human swapped your classification label. Re-read the rubric and the issue, work out which criterion (or the scope guard or budgets) would have given their answer, and write `artifacts/rubric-amendment-template.md` to a scratch file: quote the current wording exactly as it appears in `artifacts/rubric.md`. Then `python3 -m apm.rubric propose <issues_dir> <id> <scratch_file>` and wait. Never a second amendment on the same issue.
   - `[amend]`: the human replied to your amendment. `python3 -m apm.rubric decide <issues_dir> <id> artifacts/rubric.md` applies it (version bump) or records the rejection.
4. Finish with `python3 -m apm.triage digest <issues_dir> <started>` and show the digest to the operator verbatim.

## Never

- Post a second brief, question set, or proposal on an issue the human has not commented on since your last one.
- Create sub-issues other than through `apm.approval approve`, set a status to done, or touch labels other than through `apply` and `approve`.
- Read sources outside the context pack unless the issue links them.

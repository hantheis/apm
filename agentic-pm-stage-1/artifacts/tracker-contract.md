# Tracker contract

_Portable core. Version 1, 2026-09-12._

Every tracker touchpoint in the agentic PM system goes through one thin boundary. This document is the contract that boundary exposes. Any tracker implementation (local markdown, Linear, Buganizer) must satisfy it; nothing in the workflow protocol may reach around it.

The contract exposes exactly the operations the workflow protocol needs and no more. An operation a tracker happens to offer but the protocol does not need is a convenience, not part of the contract, and must not be added here.

## Concepts

- **Issue.** The unit of work and the only record. Has an **id** (opaque string, unique in its tracker), a **title**, a **body** (free text; once approved, the brief lives here), a **status**, a set of **labels**, an optional **parent** issue, a list of **blocked-by** issues, and an ordered list of **comments**.
- **Status.** A single value from the tracker's own status vocabulary. Some trackers have a fixed set (Buganizer: New, Assigned, Accepted, Fixed, Verified, and so on); some are free text. The protocol never names a raw status. It names two abstract states, and each implementation maps them: **done** (the set of tracker statuses that mean the work is finished; the human sets one of these, the agent never does) and **new** (the status a freshly created sub-issue gets). Everything else the workflow needs to know is carried by labels.
- **Label.** A plain string. Workflow state is carried by labels with the `apm:` prefix (see the label vocabulary). Labels must already exist in the tracker; the boundary applies and removes them, it never creates them.
- **Comment.** An **author**, a **timestamp**, and a **body**. Comments are append-only and never edited or deleted. Dialogue lives here. Agent comments end with the signature line `-- apm` (see guarantee 3).
- **Parent.** A sub-issue points at exactly one parent. Hierarchy is convenience; the load-bearing copy of any structure is the text on the parent (the proposal and the master checklist).
- **Blocked-by.** A relation from an issue to the issues that must finish first. Convenience; the load-bearing copy is the dependencies line in the brief text.

## Operations

Ten operations and one declaration.

| Operation | Input | Output | Notes |
|---|---|---|---|
| `list_issues` | optional filters: `status`, `label`, `parent` | list of issue summaries (id, title, status, labels) | The protocol decides what counts as a candidate; the boundary only filters. |
| `read_issue` | `id` | the full issue: title, body, status, labels, parent, blocked-by, comments in order | Always the current state. The protocol reads its own prior comments and labels from here before acting. |
| `post_comment` | `id`, `author`, `body` | the comment as recorded (with timestamp) | Append-only. |
| `add_label` | `id`, `label` | none | Idempotent: adding a label the issue already carries changes nothing. |
| `remove_label` | `id`, `label` | none | Idempotent: removing an absent label changes nothing. |
| `create_issue` | `title`, `body`, optional initial `labels` | the new issue's id | A root issue at the tracker's new-issue status, untriaged. An operator-side write: one-line capture and filed next steps use it; the protocol never creates work on its own. |
| `create_sub_issue` | `parent_id`, `title`, `body`, optional initial `labels` | the new issue's id | Sets the parent. Called only after approval. |
| `add_blocked_by` | `id`, `blocker_id` | none | Idempotent. |
| `read_status` | `id` | the status string | |
| `set_status` | `id`, `status` | none | `status` must be one the tracker has; an implementation with a fixed vocabulary rejects anything else. The agent never sets a done status. |
| `done_statuses` | none | the set of this tracker's statuses that mean done | A declaration, not a call. The protocol tests `status in done_statuses`; it never compares against literal status names. |

## Guarantees every implementation must give

1. **Read-after-write.** A `read_issue` after any write reflects that write.
2. **Idempotent label and relation writes.** Repeating `add_label`, `remove_label`, or `add_blocked_by` with the same arguments is a no-op.
3. **Comments are ordered and attributable.** `read_issue` returns comments in posting order with author and timestamp. The protocol must be able to find its own: a comment is the agent's if its author is the agent identity **or** its body ends with the signature line `-- apm`. The agent signs every comment it posts, on every tracker, because some trackers (Linear through the connector) attribute the agent's writes to the operator.
4. **No hidden state.** Everything the protocol wrote is visible through `read_issue`. There is no side channel.
5. **No creation of labels or statuses.** Set-up of the vocabulary is an operator step, written down per implementation.
6. **Status mapping is declared, not assumed.** Every implementation states its `done_statuses` and its new-issue status in its own setup notes. Protocol code that mentions a literal status name is a bug.

## What is deliberately not here

- Editing or deleting comments, issues, or labels. The record is append-only.
- Assignees, priorities, estimates, projects, cycles, teams. If the protocol needs one of these later it is added to the contract first, then to every implementation.
- Searching issue text. Prior-art search is done by the protocol over `list_issues` plus `read_issue`, bounded by the pass budget.
- Webhooks or event subscriptions. Stage 2 delivers events to a handler that then calls these same operations.

## Implementations

- **Local markdown** (test seam, v1): `apm/tracker/local_markdown.py`, file convention in `docs/agents/issue-tracker.md`.
- **Linear** (v1 end-to-end): `trackers/linear.md`, the connector-call mapping the agent follows. Verified 2026-09-12.
- **Buganizer** (destination): the port. Steps and findings so far in `port-log.md`.

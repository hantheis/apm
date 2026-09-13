# Setup checklist

_Portable core. Version 2, 2026-09-13 (staged carry-over list added to section 5). Everything an adopter does to install the system and make it theirs, in one sitting. Follow it top to bottom; the files it names are the only ones you edit. Lines marked **[Antigravity]** say where the Antigravity shell will differ from the Claude Code one; nothing is built for that yet._

## What you are installing

Two halves and a verification set (`PACKAGE.md` lists every file):

- **Portable core**: `artifacts/` (the text: contracts, protocol, rubric, templates, packs, rules) and `apm/` (the mechanical steps of the protocol, automated for the local-markdown tracker, plus the checks that keep the agent honest). Agent-agnostic.
- **Claude Code shell**: `skills/triage-queue/` and `skills/execute-task/`. Thin glue that tells Claude Code which core file to read and which command to run at each step. It states no logic of its own; `python3 -m apm.package check` enforces that.
- **Verification**: `fixtures/` (scenario issue sets) and `tests/`.

## 1. Install (about ten minutes)

1. Requirements: Python 3.11 or later (standard library only), git, Claude Code.
2. Copy or clone the repository. From its root:

   ```bash
   python3 -m unittest discover -s tests
   ```

   Every test passes on a fresh copy.
3. Install the shell. Project-level (this repository only):

   ```bash
   python3 -m apm.package install
   ```

   or user-level, so the skills are available in every Claude Code session:

   ```bash
   python3 -m apm.package install --user
   ```

   Either creates symlinks; re-running is a no-op. Claude Code then offers `/triage-queue` and `/execute-task`.
4. Confirm the package is consistent:

   ```bash
   python3 -m apm.package check
   ```

   It fails if a skill names a core file or command that does not exist, uses a label the vocabulary does not define, states a budget number, or if this checklist misses a customization point.

**[Antigravity]** Step 3 differs. Antigravity reads its own rules, workflows, and skills files; copy the body of each `skills/*/SKILL.md` into that format. The steps inside stay word for word, because they only cite core files and boundary operations. Step 4's check still runs on the core.

## 2. Make it yours

Every file you are expected to edit, and what it controls. Nothing else needs editing to customize the system; the rest of `artifacts/` is protocol, contracts, and worked examples.

| File | Controls | Do this |
|---|---|---|
| `artifacts/context/operator.md` | Who the operator is; the standing rule-sets every brief inherits; defaults (dates, spelling, comms are drafts). | Replace the TDL details with yours. Cite each rule-set by path. |
| `artifacts/rules/` | Standing rule-sets. Hard rules bind executing agents on par with the rubric and are ticked in every pre-delivery self-check; soft rules are defaults a brief may override. Two ship: `tdl-house-style.md` (how an output reads and what it cites) and `output-quality.md` (whether it is finished). A rule that code already refuses goes under a third heading, "Enforced by deliver", so it binds without being ticked. | Edit either, or add your own file with the same headings, then cite it in `operator.md`. Keep hard rules few; each one is a line the agent must tick on every delivery. |
| `artifacts/context/` | One context pack per client or project: glossary, stakeholders, sources, delegated document types, deliverables folder. The pack is the agent's primary source and its cross-client boundary. | Write `<slug>.md` per project from `context-pack-format.md` (`northwind.md` is the worked example; delete it when you have a real one). Label that project's issues `project:<slug>`. |
| `artifacts/templates/` | The shape of each deliverable type: required sections in order, forbidden phrases, style, skeleton. | Edit the core files in place, or hand the system a sample of your own format: `python3 -m apm.templates adopt <sample.md> <kind> [--project <slug>]`. See `templates/README.md`. |
| `artifacts/rubric.md` | How issues are classified (five criteria, scope guard) and the budget defaults (turn, tool-call, depth, loop, pass, prior-art caps). | Edit the wording and the numbers directly and bump the version line. After that, let the amendment loop propose changes when you overrule a classification. |
| `artifacts/labels.md` | The meaning of the eight `apm:` labels and the `project:` marker. | Read it; create the eight labels in your tracker with these descriptions. The names are fixed (the code and the skills use them); do not rename. |
| `artifacts/trackers/` | Tracker configuration and the mapping of the ten contract operations onto a real tracker. | Local markdown needs nothing. For Linear, follow the operator-setup steps in `linear.md` and record your team id there. For another tracker, write `<tracker>.md` from `tracker-contract.md`: scope, the labels, its `done_statuses`, the new-issue status, and the agent identity. |
| `artifacts/stores/` | Where deliverables go. | v1 uses a local directory (the `<store_dir>` argument to `/execute-task`). Google Docs is specified in `google-docs.md` and configured at port time: fill in its configuration block. |
| `apm/tracker/contract.py` | The agent's author name and comment signature (`AGENT_AUTHOR`, `SIGNATURE`). | Leave as `apm` / `-- apm` on any tracker that writes as the operator. Change only when porting to a tracker where the agent has its own identity (then the signature is dropped; see `port-log.md`). |

Hand `artifacts/replying.md` to anyone who will review the agent's work: it lists the reply phrases the protocol understands.

**[Antigravity]** The same files, unchanged. Two differ in content, not shape: `trackers/buganizer.md` (to be written from the contract; the port log's right-hand column has the Buganizer equivalents, including the fixed status set, hotlists for labels, and the open project-marker question) and `stores/google-docs.md` (filled in by the Antigravity agent per its setup steps).

## 3. First run

Ticket 02's scenario on the local tracker: one agent-capable issue becomes a brief.

```bash
python3 -m apm.fixtures triage-agent-capable
```

That prints the path of a working copy. In Claude Code, run `/triage-queue <that path>` and expect: one `#### Brief` comment on issue 01, label `apm:agent`, and a digest with one item under Triaged. Run it again and expect the digest to show nothing new; the pass is idempotent.

Without Claude Code, the mechanical steps alone reproduce the pass, using the recorded brief. The fixture issue carries no `project:` label, so name `northwind` as the default pack:

```bash
python3 -m apm.triage plan <copy>
```

```bash
sed -n '/^#### Brief/,$p' artifacts/brief-example.md > brief.md
```

```bash
python3 -m apm.triage apply <copy> 01 agent brief.md artifacts/context northwind
```

```bash
python3 -m apm.triage digest <copy> <the started: timestamp plan printed>
```

A second `plan` prints no candidates.

## 4. Daily use

- `/triage-queue <issues_dir> [default_pack]`: one pass over the queue. Run it as often as you like; it changes nothing it has already handled.
- `/execute-task <issues_dir> <store_dir> <id>`: execute one approved agent sub-issue from its brief. The delivery comment ends with the agent's self-check: every acceptance criterion and hard rule ticked with a reason. Read it as what was verified; your review starts from there.
- File tasks with the six lines in `artifacts/intake-template.md` (project, make, from, done when, notes). Most needs-info questions come from a missing line there.
- Capture in one line when you are not at the keyboard for long: `python3 -m apm.capture <issues_dir> "<title>" --project <slug> [--make ...]` files the issue with the template pre-filled and the `project:` label applied; the pass asks for the rest. On Linear, save the six lines as the team's issue template and bind a shortcut to "new issue".
- Reply on issues in the phrases `artifacts/replying.md` lists (approve, re-decompose, leaf N is mine, the comments are in the Doc, file them). The next pass acts on them.

**[Antigravity]** The same two entry points as Antigravity workflows. The design summary expects Buganizer's assign-to-agent machinery to replace `/execute-task` entirely; confirm on the first real task and record the answer in `port-log.md`.

## 5. Porting

`artifacts/port-log.md` is the running record of every setup step taken on Linear with its corporate equivalent, and the list of decisions still open at port time. Read it before repeating the setup elsewhere, and append to it as you go.

### What to carry over, in stages

Bring the core in the order the first real issue needs it. Each stage is complete on its own; nothing in a later stage is required for the earlier one to work. Leave out `context/` packs for real clients on this side, the `.scratch/` directory, and the `.claude/` symlinks.

**Stage 1: triage one real issue.** The agent reads the rubric and posts a brief, a proposal, or questions. Nothing is executed or stored.

- `rubric.md`, `labels.md`, `workflow-protocol.md`, `replying.md`
- `brief-template.md`, `proposal-template.md`, `intake-template.md`, `context-pack-format.md`
- `context/operator.md` rewritten for the new operator, plus one context pack for the first project
- `rules/output-quality.md`, and the house-style file if there is one
- `skills/triage-queue/SKILL.md`, as the source of the triage workflow in the target agent's format
- `tracker-contract.md`, and `trackers/<tracker>.md` written from it at this stage. Create **all eight** `apm:` labels now, even though triage uses four: retrofitting labels was gap 2 on Linear and cost a pass.

Skip the `*-example.md` files at first; they are half the core's length and matter only when the agent's output looks wrong. Skip `apm/` entirely: on a tracker where the agent performs the boundary operations itself there is no seam for the Python to run against, and writing a tracker implementation is not a stage 1 job.

**Stage 2: execute and deliver.** Add when a triaged issue is agent-capable and the output is wanted.

- `skills/execute-task/SKILL.md`, `templates/`, `deliverable-examples.md`, `delivery-example.md`, `stuck-template.md`
- `artifact-store-contract.md` and `stores/google-docs.md`, with its setup steps 1 to 3 run and recorded
- `apm/templates.py` and `apm/context.py` as reference for the template check and the isolation check, if the target agent can run them on a delivered draft

**Stage 3: the loop.** Add once the first delivery has been reviewed: `review-example.md`, `approval-example.md`, `rubric-amendment-template.md` and its example, the remaining examples, and `port-log.md` itself.

Everything else in the manifest (`trackers/linear.md`, `apm/tracker/`, `apm/artifacts/`, `fixtures/`, `tests/`) stays behind unless the target agent ends up running `apm/` against a real tracker implementation; then bring the whole manifest and run the tests on the copy first.

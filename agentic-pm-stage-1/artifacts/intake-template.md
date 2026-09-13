# Intake template

_Portable core. Version 1, 13 September 2026. What to write when you file a task so the first pass can brief it instead of asking. Six lines; most tasks need four; two are enough to capture (see Capture below). Everything the agent needs beyond these lines comes from the project's context pack (`context/<slug>.md`), so keep the packs current and the issues short._

## The template

Paste into the issue body. Delete any line you do not need except **Project**, **Make**, and **From**.

```markdown
**Project:** <slug>
**Make:** <email draft | status update | research | PRD | document | comment> for <who reads it>
**From:** <link or pack source name>: <what it is for>
**From:** <second source, if any>
**Done when:** <one line a reviewer can tick from the output alone>
**Notes:** <deadline, tone, exclusions, who not to mention, or "none">
```

## What each line is for

| Line | Feeds | Without it |
|---|---|---|
| **Project** | The `project:<slug>` label and the context pack: stakeholders, sources, rules, deliverables folder, glossary. | Needs-info, always. No pack, no brief. |
| **Make** | The output type (picks the template) and the reader (sets voice and scope). Rubric criterion 3. | Needs-info, or the wrong template. |
| **From** | The brief's Inputs. Rubric criterion 2. A link, or the exact name of a source the pack lists ("Working notes"), one per line, with what it is for. | Needs-info. |
| **Done when** | The first acceptance criterion. Rubric criterion 1. Optional when the type implies it (an email recapping a meeting; a fortnightly status update). | Guessed from the type; you get a question if the type does not imply it. |
| **Notes** | Anything the pack does not say: a deadline, an exclusion, a tone to match, a person to copy. | Fine to leave out. |

Nothing else is needed. Do not list stakeholders, rules, budgets, or folders in the issue: the pack has them. If the pack is missing something you keep typing into issues, add it to the pack once.

## Capture: the two-line minimum

You do not have to fill the template at the moment a task occurs to you. **Project** plus **Make** is a valid capture: file it, and the first pass asks for what is missing as a `#### Questions` comment (needs-info). Answer by reply and the next pass briefs it. A capture with **From** as well usually briefs without a question.

On the local tracker, one line files it with the template pre-filled and the `project:` label applied:

```bash
python3 -m apm.capture <issues_dir> "Follow-up email to Priya" --project northwind --make "email draft for Priya Raman"
```

Add `--from`, `--done-when`, `--notes` when you have them; `--no-project` files it unlabelled and the pass asks which project. The command prints what the pass will ask for. On Linear and Buganizer the equivalent is the issue template below, plus a keyboard shortcut or phone shortcut that opens a new issue from it.

The trade: a thin capture costs one question-and-answer round before it is briefed. The six lines cost the same thought up front and skip the round. Capture when the idea would otherwise be lost; fill the template when you are already at the keyboard.

## Filed next steps

When a delivery ends with `#### Next steps` and you reply `file them`, each item arrives as an issue in this same form: **Project** from the delivered issue, **Make** from the item's one line, **From** naming the delivery it follows on from, **Notes** carrying the suggested kind. It is triaged like anything you filed by hand. Add a **From** line to it if the follow-on needs a source the delivery did not name.

## Rules of thumb

- **A link beats a description.** Paste the Doc, folder, or ticket. Put links in the issue body, not the tracker's attachments field; the protocol reads the body.
- **Name people as the source names them.** The agent cross-checks names against the pack and the linked sources; a mismatch becomes a question or a flagged gap in the draft.
- **One output per issue.** A task that wants two documents is two issues, or say "this is big, split it" in Notes and the agent proposes a decomposition instead of asking.
- **Say "draft".** Anything that would be sent, shared, or published is a draft for you to send; the agent will not do the sending, so do not ask it to.
- **If the person is new, add them to the pack**, not the issue. One line in Stakeholders.

## Examples

**Enough to brief straight away** (the follow-up email as it should have been filed):

```markdown
**Project:** northwind
**Make:** email draft for Priya Raman
**From:** https://docs.google.com/document/d/<doc id>: notes and transcript of the sponsor sync on 2 September 2026
**Done when:** recaps what we agreed and lists everyone's next steps with owners
**Notes:** copy the design lead; keep it short and informal
```

**Enough to decompose straight away** (a big one; the pass proposes leaves instead of asking):

```markdown
**Project:** northwind
**Make:** document for Hannah; this is big, split it
**From:** Working notes: the last three sponsor syncs, for what Priya has asked for
**From:** Drive:/TDL/Northwind/Proposals: the original engagement proposal, for the agreed scope
**Done when:** Northwind has a one-page plan for the onboarding refresh with owners and dates
**Notes:** first draft by 30 September 2026
```

**Still needs-info** (nothing to brief from):

```markdown
Create Q4 expectations
```

## Where it lives per tracker

- **Local markdown:** the body of `NN-<slug>.md` below the title.
- **Linear:** save the six lines as an issue template on the team (Team settings, Templates) so every new issue starts with them; the `project:` line is a reminder to add the label, which the template can also pre-set.
- **Buganizer:** a bug template on the component; the `project:` line maps to whatever carries the project marker there (`port-log.md`).

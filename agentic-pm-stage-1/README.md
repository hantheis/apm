# agentic pm, stage 1

The portable core of the agentic PM system, cut down to what the first real issue needs: the agent reads the rubric and posts a brief, a proposal, or questions. Nothing is executed or stored at this stage.

Exported 13 September 2026 from the v1 build (Linear, Claude Code). Stages 2 and 3 are listed in `artifacts/setup-checklist.md`, section 5.

## Read first

1. `artifacts/setup-checklist.md`: sections 1, 2, and 5. Section 5 says what this stage contains and what comes later.
2. `artifacts/port-log.md`: the decisions already taken for Buganizer (project marker is the bug hierarchy; assignment means triage, then execute if agent-capable) and the setup steps taken on Linear with their corporate equivalents.
3. `artifacts/tracker-contract.md`: the ten operations to map onto Buganizer. Write `artifacts/trackers/buganizer.md` from it, the way the Linear mapping was written.

## To do on this side

- Rewrite `artifacts/context/operator.md` for the corporate operator. `artifacts/context/northwind.md` is a fictional worked example of a project pack; replace it with one pack for the first real project.
- `artifacts/rules/tdl-house-style.md` is the previous operator's house style, kept as the shape of a rule-set; replace or drop.
- Create all eight `apm:` hotlists from `artifacts/labels.md` before the first pass.
- Turn `skills/triage-queue/SKILL.md` into an Antigravity workflow. The steps only cite core files, so they port word for word.

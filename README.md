# Outcome OS — AI Harness

Outcome OS is a local-first execution controller for one recurring problem: **AI can produce work, but a human still has to keep restating the goal, checking whether it is truly done, and deciding what happens next.**

It turns a goal into an auditable operating loop:

1. define measurable completion criteria;
2. import or add a prioritized work queue;
3. execute one item at a time;
4. attach concrete evidence and validation checks;
5. verify the whole original goal—not just the latest response;
6. stop only when every completion gate passes.

The system is standard-library Python, has no hosted service, and keeps its state locally under `.outcome-os/`.

## Why this exists

The surrounding portfolio already contains strong specialized layers:

- **Portfolio OS** can inspect repositories and generate an evidence-backed backlog;
- **ChatGPT Goals** can alternate work and verification turns;
- **PM Command Center** can generate project-management artifacts;
- **Medusa** is being hardened through issue → implementation → CI → merge loops.

Outcome OS is the missing control plane. It provides the shared definition of done, state machine, evidence ledger, deterministic completion gate, escalation model, and portable prompts that bind those tools together.

## Install

```bash
python -m pip install -e .
```

Or run directly:

```bash
python outcome_os.py --help
```

## Start a goal

```bash
mkdir medusa-outcome && cd medusa-outcome
outcome-os init "Medusa issue completion" \
  --objective "Take one unclaimed Medusa issue from selection through verified merge and closure" \
  --repo benclawbot/Medusa \
  --criterion "The selected issue has an implementation PR" \
  --criterion "All required CI and repository validation pass" \
  --criterion "The PR is merged into main" \
  --criterion "The linked issue is correctly closed with evidence"
```

Add or import work:

```bash
outcome-os add-item "Select the highest-value open issue without a PR" --priority 100
outcome-os add-item "Implement the bounded change" --priority 90
outcome-os import-portfolio ~/.portfolio-os/backlog/backlog.json
```

Record execution:

```bash
outcome-os set-item w-12345678 active
outcome-os run-check unit-tests "cargo test --workspace"
outcome-os evidence c-12345678 "PR #603 implements the accepted scope" --type pull-request --source github
outcome-os set-item w-12345678 done --note "Merged after required checks passed"
```

Verify the entire goal:

```bash
outcome-os verify
```

Exit code is `0` only when all completion gates pass; incomplete verification returns `2`.

## AI work and verification prompts

```bash
outcome-os prompt work_prompt
outcome-os prompt verify_prompt
```

These prompts are designed to be pasted into ChatGPT Goals or another agent runner. They always reintroduce the full goal, completion criteria, current queue, evidence requirements, remaining work, and verifier-selected next action.

## Dashboard and integrity check

```bash
outcome-os dashboard
outcome-os doctor
```

The dashboard is a self-contained local HTML file with no CDN, trackers, external fonts, or network requests. `doctor` verifies the append-only SHA-256 event chain and structural integrity of the current state.

## Completion gate

A goal is complete only when:

- every criterion has its minimum concrete evidence;
- all checks required by criteria pass;
- every work item is `done` or explicitly `skipped`;
- no blocker remains open;
- aggregate confidence meets the configured threshold.

A prose claim such as “done” does not affect the gate.

## Portfolio OS bridge

`import-portfolio` accepts common Portfolio OS backlog shapes and normalizes stable IDs, priorities, titles, and acceptance criteria into the Outcome OS queue. Re-importing the same stable item is idempotent.

This creates a clean responsibility boundary:

```text
Portfolio OS      → discover and prioritize evidence-backed work
Outcome OS        → own goal state, ordering, evidence, verification, escalation
AI/ChatGPT agent  → perform the current action
GitHub/CI         → provide external implementation and test evidence
PM Command Center → produce governance artifacts when a stage requires them
```

## Repository contents

```text
outcome_os.py                  CLI, state machine, ledger, verifier, dashboard
examples/medusa-goal.json      Ready-to-use Medusa completion goal
examples/portfolio-backlog.json Import example
scripts/run-medusa-loop.sh     Human/agent execution loop
scripts/demo.sh                Reproducible local demonstration
docs/OPERATING_MODEL.md        Roles, gates, escalation, weekly workflow
docs/GOAL_DESIGN.md            How to write verifiable goals
.github/workflows/ci.yml        Cross-platform syntax, test, package checks
tests/test_outcome_os.py       Deterministic unit tests
```

## Safety model

Outcome OS does not bypass authentication, tool permissions, protected branches, reviews, safety policies, or confirmation requirements. It records blockers instead of silently treating inaccessible work as complete. Destructive actions remain outside the automatic core.

## Development

```bash
python -m compileall -q outcome_os.py
python -m unittest discover -s tests -v
python -m build
```

## License

MIT

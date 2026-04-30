# Orchestration Loop

The orchestration loop is the execution engine of Open Agentic CMO.

It is owned by Porky, the orchestrator and validation gatekeeper.

The loop is responsible for continuously scanning workflow state, validating signals and artifacts, determining the next executable phase, and advancing the workflow only when it is safe to do so.

---

## Why the orchestration loop matters

Most multi-agent systems fail because the orchestrator behaves passively.

Common failures include:

- Stopping after one agent completes
- Waiting for manual intervention even when the next step is ready
- Advancing based on vague comments
- Missing signals posted in subtasks
- Ignoring artifacts already produced
- Re-running completed phases
- Creating duplicate outputs
- Letting downstream agents run too early

Open Agentic CMO avoids these failures through a continuous, signal-driven, artifact-aware orchestration loop.

---

## Core principle

The loop follows one rule:

> After any action, recompute the workflow state and determine the next safe action.

Porky does not assume the workflow is complete after a signal appears.

Porky does not assume a phase is complete because a task status changed.

Porky only trusts:

- valid signals
- valid artifacts
- verified downstream state

---

## Loop trigger events

The orchestration loop must run after any meaningful workflow event.

Examples:

- Parent issue is created
- Subtask is created
- Agent is delegated
- Signal is emitted
- Artifact is published
- Artifact is corrected
- A phase is blocked
- A timeout is detected
- Recovery is attempted
- Delivery is completed

After each event, Porky must re-scan the workflow from scratch.

---

## Loop sequence

At every loop iteration, Porky performs these steps:

1. Scan the parent issue.
2. Scan all subtasks.
3. Extract all SIGNAL blocks.
4. Extract all ARTIFACT blocks.
5. Validate signal format.
6. Validate artifact structure.
7. Normalize valid signals and artifacts to the parent workflow.
8. Deduplicate signals and artifacts.
9. Recompute workflow state.
10. Validate the current phase gate.
11. Determine the next executable phase.
12. Execute the next safe action.
13. Repeat.

The loop stops only when the workflow is complete or blocked.

---

## Loop diagram

```text
Start / Event
    ↓
Scan parent issue
    ↓
Scan subtasks
    ↓
Extract signals + artifacts
    ↓
Validate signals + artifacts
    ↓
Normalize to parent workflow
    ↓
Recompute workflow state
    ↓
Is next phase valid?
    ├── Yes → Execute next phase → Repeat loop
    └── No  → Block or wait for required artifact/signal
```

---

## Workflow state model

Porky maintains an explicit workflow state.

The state should include:

- current_phase
- completed_phases
- pending_phase
- last_valid_signal
- validation_status
- known_artifacts
- blocked_reason
- next_expected_signal
- next_expected_artifact

State must be recomputed from actual workflow evidence.

Porky should not rely on memory alone.

---

## Phase state

Each phase may be in one of several logical states.

| State | Meaning |
|---|---|
| Not started | No valid signal or artifact exists |
| In progress | Work has been delegated but no valid artifact exists yet |
| Artifact present | Artifact exists but still needs validation |
| Signal present | Signal exists but artifact must still be checked |
| Validated | Signal and artifact are valid |
| Blocked | Missing or invalid requirement prevents progress |
| Complete | Phase is validated and downstream-safe |

Only validated or complete phases can unlock downstream execution.

---

## Phase transition model

The workflow advances in this order:

1. Research
2. Strategy
3. Content
4. Notion
5. Delivery
6. Completion

Each transition requires a valid previous phase.

---

## Research phase

Required output:

- `audit` artifact
- `AUDIT_READY` signal

Owner:

- Babe

Porky may advance to Strategy only when:

- the audit artifact exists
- the audit artifact passes validation
- `AUDIT_READY` exists or is reconciled from a valid audit artifact

---

## Strategy phase

Required output:

- `strategy` artifact
- `SYNTHESIS_READY` signal

Owner:

- Porky

Porky may advance to Content only when:

- the strategy artifact exists
- the strategy artifact passes validation
- `SYNTHESIS_READY` exists or is reconciled from a valid strategy artifact

---

## Content phase

Required output:

- `content_set` artifact
- `CONTENT_SET_READY` signal

Owner:

- Hamm

Porky may advance to Notion only when:

- the content_set artifact exists
- the content_set artifact passes validation
- `CONTENT_SET_READY` exists or is reconciled from a valid content_set artifact
- the audit artifact is still available for Pumba to persist Research Babe

---

## Notion phase

Required output:

- Notion Content Pipeline entries
- Research Babe page
- `NOTION_SYNC_COMPLETE` signal

Owner:

- Pumba

Porky may advance to Delivery only when:

- all content items exist in Notion
- content is stored in page body, not Notes
- required Notion fields are populated
- no duplicates exist
- Research Babe page exists
- Research Babe page includes required sections
- `NOTION_SYNC_COMPLETE` exists and passes validation

---

## Delivery phase

Required output:

- Telegram summary
- `DELIVERY_COMPLETE` signal

Owner:

- Pumba

Porky may complete the workflow only when:

- Telegram summary was sent once
- summary references content persistence
- summary references Research Babe
- no duplicate delivery exists
- `DELIVERY_COMPLETE` exists and passes validation

---

## Stop conditions

The loop stops only in two cases.

### 1. Workflow complete

The workflow is complete when:

- all phases passed validation
- `DELIVERY_COMPLETE` exists
- Notion state is verified
- Research Babe exists
- Telegram delivery is verified
- no corrupted or duplicate output exists

### 2. Workflow blocked

The workflow blocks when:

- a required signal is missing
- a required artifact is missing
- an artifact is invalid
- Notion state is corrupted
- Research Babe is missing
- delivery is incomplete
- downstream execution would be unsafe

A blocked workflow is preferable to corrupted output.

---

## What the loop must not do

Porky must not:

- advance based on summaries
- advance based on task status alone
- advance based on agent claims
- skip artifact validation
- run Hamm before strategy is valid
- run Pumba before content_set is valid
- run Pumba if the audit artifact is missing
- complete the workflow before delivery is verified
- duplicate content
- duplicate Notion pages
- duplicate Telegram summaries

---

## Downstream execution rule

Downstream agents must not execute early.

Hamm may run only after:

- audit artifact is valid
- strategy artifact is valid
- `SYNTHESIS_READY` exists or is reconciled

Pumba may run only after:

- audit artifact is valid
- content_set artifact is valid
- `CONTENT_SET_READY` exists or is reconciled

This prevents noise, premature `BLOCKED` signals, and wasted execution.

---

## Artifact-first execution

The loop assumes artifacts are the source of truth.

Correct order:

1. Artifact is published.
2. Artifact is validated.
3. Signal is emitted.
4. Signal is propagated.
5. Porky advances the workflow.

Incorrect order:

1. Signal is emitted.
2. Artifact is missing.
3. Downstream phase starts.

The incorrect order must block.

---

## Reconciliation behavior

The orchestration loop can recover from partial execution.

### Artifact exists, signal missing

If a valid artifact exists but the signal is missing, Porky may:

1. Validate the artifact.
2. Register internal phase completion.
3. Request signal propagation.
4. Continue safely.

### Signal exists, artifact missing

If a signal exists but the artifact is missing, Porky must:

1. Treat the signal as incomplete.
2. Block the workflow.
3. Request the missing artifact.
4. Prevent downstream execution.

### Artifact invalid

If an artifact exists but fails validation, Porky must:

1. Block the workflow.
2. Identify the responsible agent.
3. Explain the invalid requirement.
4. Request correction.

---

## Idempotency behavior

The loop must avoid duplicate work.

Before executing any phase, Porky checks:

- Has this phase already completed?
- Does the artifact already exist?
- Is the artifact valid?
- Has downstream persistence already occurred?
- Would executing now create duplicates?

If valid completion exists, Porky skips execution and continues the loop.

Examples:

- Do not rerun Babe if a valid audit exists.
- Do not rerun Hamm if a valid content_set exists.
- Do not rerun Pumba if Notion is already correct.
- Do not resend Telegram if delivery already happened.

---

## Blocking behavior

When Porky blocks the workflow, the block must be actionable.

A block should include:

- blocked phase
- responsible agent
- missing or invalid requirement
- required next action

Example:

```text
Blocked phase: Content
Responsible agent: Hamm
Issue: CONTENT_SET_READY was emitted but no full content_set artifact exists.
Required next action: Hamm must publish the full content_set artifact before emitting CONTENT_SET_READY again.
```

---

## Observability

At every transition, Porky should log:

- current phase
- completed phases
- pending phase
- validated signal
- validated artifact
- validation result
- action taken
- next expected signal
- next expected artifact
- blocked reason, if any

These logs help humans understand why the workflow advanced or blocked.

---

## Example successful loop

A successful workflow may look like this:

```text
1. Porky scans parent issue.
2. No AUDIT_READY found.
3. Porky delegates research to Babe.
4. Babe publishes audit artifact.
5. Babe emits AUDIT_READY.
6. Porky validates audit.
7. Porky publishes strategy artifact.
8. Porky emits SYNTHESIS_READY.
9. Porky delegates content to Hamm.
10. Hamm publishes content_set artifact.
11. Hamm emits CONTENT_SET_READY.
12. Porky validates content_set.
13. Porky delegates persistence to Pumba.
14. Pumba creates Content Pipeline pages.
15. Pumba creates Research Babe page.
16. Pumba emits NOTION_SYNC_COMPLETE.
17. Porky validates Notion state.
18. Pumba sends Telegram summary.
19. Pumba emits DELIVERY_COMPLETE.
20. Porky validates delivery.
21. Porky posts final summary.
22. Workflow complete.
```

---

## Example blocked loop

A blocked workflow may look like this:

```text
1. Hamm emits CONTENT_SET_READY.
2. Porky scans Hamm subtask.
3. Porky finds signal.
4. Porky looks for content_set artifact.
5. No full artifact exists.
6. Porky marks CONTENT_SET_READY as incomplete.
7. Porky blocks workflow.
8. Porky requests Hamm to publish full content_set artifact.
9. Pumba is not allowed to run.
```

This prevents empty or incomplete content from being persisted into Notion.

---

## Summary

The orchestration loop is what turns Open Agentic CMO from a set of agents into a reliable system.

It ensures that every phase is:

- explicit
- validated
- recoverable
- idempotent
- safe for downstream execution

The loop is strict by design.

In agentic workflows, strictness is what prevents silent failure.

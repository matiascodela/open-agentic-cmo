# Orchestration Loop

The orchestration loop is the execution engine of Open Agentic CMO.

It is owned by Porky, the orchestrator and validation gatekeeper.

The loop is responsible for continuously scanning workflow state, validating signals and artifacts, determining the next executable phase, creating complete phase-specific subtasks, and advancing the workflow only when it is safe to do so.

---

## Why the orchestration loop matters

Most multi-agent systems fail because the orchestrator behaves passively or delegates too early.

Common failures include:

- Stopping after one agent completes
- Waiting for manual intervention even when the next step is ready
- Advancing based on vague comments
- Missing signals posted in subtasks
- Ignoring artifacts already produced
- Re-running completed phases
- Creating duplicate outputs
- Creating all downstream subtasks at workflow initialization
- Creating title-only subtasks with missing context
- Letting Hamm run before Babe and Porky finish research + strategy
- Letting Pumba run before Hamm produces valid content
- Treating early downstream `BLOCKED` signals as real workflow failures
- Completing the workflow without parent issue visibility

Open Agentic CMO avoids these failures through a continuous, signal-driven, artifact-aware, sequential orchestration loop.

---

## Core principle

The loop follows one rule:

> After any action, recompute the workflow state and determine the next safe action.

Porky does not assume the workflow is complete after a signal appears.

Porky does not assume a phase is complete because a task status changed.

Porky does not assume downstream agents should run just because their eventual phase exists.

Porky only trusts:

- valid signals
- valid artifacts
- verified downstream state
- verified Notion state
- verified delivery state

---

## Canonical workflow order

The workflow must run sequentially:

```text
Porky → Babe → Porky → Hamm → Porky → Pumba → Porky
```

Detailed order:

1. Porky starts the parent workflow.
2. Porky delegates research to Babe only.
3. Babe completes research.
4. Babe posts the full `audit` artifact and `AUDIT_READY` signal on the original parent issue.
5. Porky validates Babe’s audit artifact.
6. Porky synthesizes the `strategy` artifact.
7. Porky emits `SYNTHESIS_READY` on the original parent issue.
8. Porky delegates content generation to Hamm only after strategy validation passes.
9. Hamm completes content.
10. Hamm posts the full `content_set` artifact and `CONTENT_SET_READY` signal on the original parent issue.
11. Porky validates Hamm’s content set artifact.
12. Porky delegates Notion persistence and delivery to Pumba only after content validation passes.
13. Pumba persists Babe’s research and Hamm’s content to Notion.
14. Pumba posts `NOTION_SYNC_COMPLETE` and `DELIVERY_COMPLETE` on the original parent issue.
15. Porky validates Notion persistence and Telegram delivery.
16. Porky posts final summary and completes the workflow.

No downstream agent should be assigned before its required upstream artifact exists and has been validated.

---

## Parent issue as source of truth

The original parent issue is the canonical workflow record.

Subtasks may exist for specialist execution, but final workflow state must be visible on the parent issue.

Required parent issue evidence:

### Babe completion

```text
ARTIFACT:
type: audit
...
```

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
...
```

### Porky strategy completion

```text
ARTIFACT:
type: strategy
...
```

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
...
```

### Hamm completion

```text
ARTIFACT:
type: content_set
...
```

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
...
```

### Pumba completion

```text
ARTIFACT:
type: notion_content_pipeline
...
```

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
...
```

```text
ARTIFACT:
type: telegram_delivery
...
```

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
...
```

If an artifact or signal exists only on a subtask, Porky may use it for reconciliation, but final completion requires parent issue visibility.

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
- A subtask completes
- A downstream agent wakes early
- A missing artifact is later posted
- A previously blocked phase becomes valid

After each event, Porky must re-scan the workflow from scratch.

---

## Loop sequence

At every loop iteration, Porky performs these steps:

1. Scan the parent issue.
2. Scan all existing subtasks.
3. Extract all `SIGNAL` blocks.
4. Extract all `ARTIFACT` blocks.
5. Validate signal format.
6. Validate artifact structure.
7. Normalize valid signals and artifacts to the parent workflow.
8. Deduplicate signals and artifacts.
9. Recompute workflow state.
10. Validate the current phase gate.
11. Determine the next executable phase.
12. If delegation is required, prepare the complete subtask description.
13. Validate that the subtask description is complete.
14. Delegate only the next valid phase.
15. Repeat until complete, safely blocked, or awaiting specialist output.

The loop stops only when:

- the workflow is complete
- the workflow is safely blocked with a clear next action
- the next required specialist has been delegated with a complete subtask description and Porky is waiting for output
- no assigned actionable work remains

The loop must not stop just because there are no new comments.

---

## Loop diagram

```text
Start / Event
    ↓
Scan parent issue
    ↓
Scan existing subtasks
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
    ├── No → Block, wait, or request correction
    └── Yes
          ↓
       Does next phase require delegation?
          ├── No → Execute Porky-owned action → Repeat loop
          └── Yes
                ↓
             Is subtask description complete?
                ├── No → Block and identify missing context
                └── Yes → Delegate next specialist only → Wait / Repeat loop
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
- next_required_agent
- next_allowed_delegation
- next_subtask_description_status

State must be recomputed from actual workflow evidence.

Porky should not rely on memory alone.

Porky should not rely on task status alone.

---

## Phase state

Each phase may be in one of several logical states.

| State | Meaning |
|---|---|
| Not started | No valid signal or artifact exists |
| Ready to delegate | Upstream requirements are valid and the next specialist can be assigned |
| Delegated | Work has been assigned with a complete subtask description |
| In progress | Specialist work is underway but no valid artifact exists yet |
| Artifact present | Artifact exists but still needs validation |
| Signal present | Signal exists but artifact must still be checked |
| Validated | Signal and artifact are valid |
| Blocked | Missing or invalid requirement prevents progress |
| Complete | Phase is validated and downstream-safe |

Only validated or complete phases can unlock downstream execution.

A future phase should not be delegated while the current phase is still in progress.

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

Porky must not create all phase subtasks upfront.

Porky must only delegate the next phase when its upstream inputs exist and have passed validation.

---

## Research phase

Required output:

- `audit` artifact
- `AUDIT_READY` signal

Owner:

- Babe

Porky may delegate Research when:

- a parent workflow exists
- no valid audit artifact already exists
- Babe has not already been delegated for this cycle

Porky may advance to Strategy only when:

- the audit artifact exists
- the audit artifact passes validation
- `AUDIT_READY` exists or is reconciled from a valid audit artifact
- the audit is visible on the parent issue or safely reconciled to it

The audit artifact must include:

- `summary`
- `key_signals`
- `inconsistencies`
- `uncertainty`
- `opportunities`
- `recommended_content_angles`
- `critical_flags`

If no critical flags exist, the audit should still include:

```text
critical_flags: []
```

---

## Strategy phase

Required output:

- `strategy` artifact
- `SYNTHESIS_READY` signal

Owner:

- Porky

Porky may create Strategy only when:

- Babe’s audit artifact exists
- the audit artifact passes validation
- critical risks have been identified or explicitly absent
- the Critical Fact / Brand Risk Gate has passed

Porky may advance to Content only when:

- the strategy artifact exists
- the strategy artifact passes validation
- `SYNTHESIS_READY` exists or is reconciled from a valid strategy artifact

If Babe’s audit contains publishable risk, the strategy must include:

- `excluded_claims`
- `safe_framing`
- `content_rules` that prevent the risky claims from being used

Porky must not generate strategy angles based on unresolved critical uncertainty.

---

## Content phase

Required output:

- `content_set` artifact
- `CONTENT_SET_READY` signal

Owner:

- Hamm

Porky may delegate Content only when:

- the audit artifact is valid
- the strategy artifact is valid
- `SYNTHESIS_READY` exists or is safely reconciled
- critical risks are blocked, excluded, or safely framed
- Hamm has not already been delegated for this cycle

Hamm may run only after Porky explicitly delegates the Content phase.

Porky may advance to Notion only when:

- the content_set artifact exists
- the content_set artifact passes validation
- `CONTENT_SET_READY` exists or is reconciled from a valid content_set artifact
- the content respects `excluded_claims`
- the content uses `safe_framing` when required
- the audit artifact is still available for Pumba to persist Research Babe

---

## Notion phase

Required output:

- Notion Content Pipeline entries
- Research Babe page
- `notion_content_pipeline` artifact
- `NOTION_SYNC_COMPLETE` signal

Owner:

- Pumba

Porky may delegate Notion only when:

- the audit artifact is valid
- the content_set artifact is valid
- `CONTENT_SET_READY` exists or is safely reconciled
- Pumba has not already been delegated for this cycle

Pumba may run only after Porky explicitly delegates the Notion + Delivery phase.

Porky may accept `NOTION_SYNC_COMPLETE` only when:

- all content items exist in Notion
- content is stored in page body, not Notes
- required Notion fields are populated
- no duplicates exist
- Research Babe page exists
- Research Babe page includes required sections
- `notion_content_pipeline` artifact exists
- `NOTION_SYNC_COMPLETE` exists and passes validation

---

## Delivery phase

Required output:

- Telegram summary
- `telegram_delivery` artifact
- `DELIVERY_COMPLETE` signal

Owner:

- Pumba

Porky may complete the workflow only when:

- Telegram summary was sent once
- summary references content persistence
- summary references Research Babe
- no duplicate delivery exists
- `telegram_delivery` artifact exists
- `DELIVERY_COMPLETE` exists and passes validation

Porky must not send the Telegram summary directly.

Pumba owns delivery.

---

## Subtask creation model

Porky must never create title-only subtasks.

Every delegated subtask must include a complete issue description with enough context for the assigned agent to execute deterministically.

A valid subtask description must include:

- `parent_issue_id`
- phase name
- assigned agent
- objective
- workflow context
- source parent brief
- input artifacts required
- output artifact required
- expected signal
- parent task source-of-truth rule
- validation criteria
- blocking conditions
- explicit constraints
- propagation requirement

If Porky cannot produce a complete subtask description, Porky must not create the subtask yet.

A subtask with only a title is invalid delegation.

When creating a subtask, Porky must include the full description in the issue body, not only in a follow-up comment.

---

## Subtask delegation order

Porky must delegate in this exact order:

```text
Babe only → Strategy by Porky → Hamm only → Pumba only → Final validation by Porky
```

### Babe subtask

Create at workflow start.

Required input:

- parent issue instructions
- target entity
- date range, if available
- content scope, if available
- platform scope, if available
- language scope, if available

Required output:

- `audit` artifact
- `AUDIT_READY`

### Hamm subtask

Create only after:

- audit artifact is valid
- strategy artifact is valid
- `SYNTHESIS_READY` is emitted or safely reconciled
- Critical Fact / Brand Risk Gate has passed

Required output:

- `content_set` artifact
- `CONTENT_SET_READY`

### Pumba subtask

Create only after:

- audit artifact is valid
- content_set artifact is valid
- `CONTENT_SET_READY` is emitted or safely reconciled
- Porky has validated the content_set artifact
- no required content fields are missing
- no excluded claims appear in the content

Required output:

- `notion_content_pipeline` artifact
- `NOTION_SYNC_COMPLETE`
- `telegram_delivery` artifact
- `DELIVERY_COMPLETE`

---

## Stop conditions

The loop stops only in one of these cases.

### 1. Workflow complete

The workflow is complete when:

- all phases passed validation
- `DELIVERY_COMPLETE` exists
- `telegram_delivery` artifact exists
- Notion state is verified
- Research Babe exists
- Telegram delivery is verified
- no corrupted or duplicate output exists
- final summary has been posted

### 2. Workflow blocked

The workflow blocks when:

- a required signal is missing
- a required artifact is missing
- an artifact is invalid
- a critical fact or brand risk is unresolved
- Notion state is corrupted
- Research Babe is missing
- delivery is incomplete
- downstream execution would be unsafe
- Porky cannot produce a complete subtask description

A blocked workflow is preferable to corrupted output.

### 3. Waiting for specialist output

The workflow may pause when:

- Porky has delegated the next valid specialist
- the subtask has a complete description
- the required artifact has not yet been produced

This is not failure.

It is the expected waiting state.

---

## What the loop must not do

Porky must not:

- advance based on summaries
- advance based on task status alone
- advance based on agent claims
- skip artifact validation
- create Hamm before audit and strategy are valid
- create Pumba before content_set is valid
- create all subtasks at workflow initialization
- create title-only subtasks
- put required context only in a later comment instead of the issue body
- run Hamm before strategy is valid
- run Pumba before content_set is valid
- run Pumba if the audit artifact is missing
- complete the workflow before delivery is verified
- duplicate content
- duplicate Notion pages
- duplicate Telegram summaries
- send Telegram directly
- persist content directly to Notion

---

## Downstream execution rule

Downstream agents must not execute early.

Hamm may run only after:

- audit artifact is valid
- strategy artifact is valid
- `SYNTHESIS_READY` exists or is reconciled
- Porky explicitly delegates the Content phase

Pumba may run only after:

- audit artifact is valid
- content_set artifact is valid
- `CONTENT_SET_READY` exists or is reconciled
- Porky explicitly delegates the Notion + Delivery phase

This prevents noise, premature `BLOCKED` signals, and wasted execution.

If Pumba wakes up early before explicit delegation, it should exit cleanly or leave a waiting note. It should not treat early wake-up as workflow failure.

---

## Artifact-first execution

The loop assumes artifacts are the source of truth.

Correct order:

1. Artifact is published.
2. Artifact is validated.
3. Signal is emitted.
4. Signal is visible on the parent issue.
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
4. Continue safely if downstream execution is safe.

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

### Artifact or signal exists only in subtask

If an artifact or signal exists only in a subtask, Porky may use it for reconciliation, but the parent issue must be updated before final completion.

---

## Idempotency behavior

The loop must avoid duplicate work.

Before executing or delegating any phase, Porky checks:

- Has this phase already completed?
- Does the artifact already exist?
- Is the artifact valid?
- Has the signal already been emitted?
- Has downstream persistence already occurred?
- Has the correct specialist already been delegated?
- Does the delegated subtask already have a complete description?
- Would executing now create duplicates?

If valid completion exists, Porky skips execution and continues the loop.

Examples:

- Do not rerun Babe if a valid audit exists.
- Do not rerun Hamm if a valid content_set exists.
- Do not rerun Pumba if Notion is already correct.
- Do not resend Telegram if delivery already happened.
- Do not create a second Hamm subtask if a valid Hamm subtask already exists.
- Do not create a second Pumba subtask if a valid Pumba subtask already exists.

If a subtask exists but has only a title or lacks required context:

- treat it as invalid delegation
- update the subtask description if possible
- otherwise block and identify the missing context

---

## Blocking behavior

When Porky blocks the workflow, the block must be actionable.

A block should include:

- blocked phase
- responsible agent
- missing or invalid requirement
- why downstream execution is unsafe
- required next action

Example:

```text
Blocked phase: Content
Responsible agent: Hamm
Issue: CONTENT_SET_READY was emitted but no full content_set artifact exists.
Why unsafe: Pumba cannot persist content that does not exist as structured content_items.
Required next action: Hamm must publish the full content_set artifact before emitting CONTENT_SET_READY again.
```

A block caused by missing subtask context should identify the missing fields.

Example:

```text
Blocked phase: Delegation
Responsible agent: Porky
Issue: Hamm subtask description is incomplete.
Why unsafe: Hamm does not have the validated strategy artifact, excluded_claims, or required output contract.
Required next action: Porky must update the Hamm subtask description before Hamm starts.
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
- next required agent
- next subtask description status
- blocked reason, if any

These logs help humans understand why the workflow advanced or blocked.

---

## Example successful loop

A successful workflow may look like this:

```text
1. Porky scans parent issue.
2. No AUDIT_READY found.
3. Porky creates Babe subtask with complete description.
4. Babe publishes audit artifact on parent issue.
5. Babe emits AUDIT_READY on parent issue.
6. Porky validates audit.
7. Porky publishes strategy artifact.
8. Porky emits SYNTHESIS_READY.
9. Porky creates Hamm subtask with complete description and strategy context.
10. Hamm publishes content_set artifact on parent issue.
11. Hamm emits CONTENT_SET_READY on parent issue.
12. Porky validates content_set.
13. Porky creates Pumba subtask with complete description and required artifacts.
14. Pumba creates Content Pipeline pages.
15. Pumba creates Research Babe page.
16. Pumba publishes notion_content_pipeline artifact on parent issue.
17. Pumba emits NOTION_SYNC_COMPLETE on parent issue.
18. Pumba sends Telegram summary.
19. Pumba publishes telegram_delivery artifact on parent issue.
20. Pumba emits DELIVERY_COMPLETE on parent issue.
21. Porky validates final delivery.
22. Porky posts final summary.
23. Workflow complete.
```

---

## Example blocked loop — missing artifact

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

## Example blocked loop — premature Pumba

A blocked or waiting workflow may look like this:

```text
1. Pumba wakes before Porky delegates Notion + Delivery.
2. Pumba sees no valid content_set artifact.
3. Pumba does not persist anything.
4. Pumba does not emit NOTION_SYNC_COMPLETE.
5. Pumba does not emit DELIVERY_COMPLETE.
6. Pumba exits cleanly or leaves a waiting note.
```

This is not a workflow failure.

The better fix is that Porky should not create Pumba before content_set validation passes.

---

## Example blocked loop — title-only subtask

A blocked workflow may look like this:

```text
1. Porky creates a Hamm subtask with only a title.
2. Hamm lacks strategy context and output requirements.
3. Porky detects the subtask description is incomplete.
4. Porky treats the delegation as invalid.
5. Porky updates the subtask description if possible.
6. If it cannot update the description, Porky blocks and identifies missing context.
```

This prevents specialist agents from guessing.

---

## Summary

The orchestration loop is what turns Open Agentic CMO from a set of agents into a reliable system.

It ensures that every phase is:

- explicit
- sequential
- validated
- context-rich
- recoverable
- idempotent
- safe for downstream execution

The loop is strict by design.

In agentic workflows, strictness is what prevents silent failure.

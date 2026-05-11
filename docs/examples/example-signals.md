# Example Signals

This example shows valid and invalid signals used by Open Agentic CMO.

Signals are explicit workflow transition markers.

A signal tells Porky that a workflow phase has reached a defined state.

However, a signal alone is not enough.

Every completion signal must map to a valid artifact or verified output.

Signals move the workflow forward only when Porky can validate the underlying artifact, producer, issue, and phase state.

The latest version adds stricter parent issue visibility, sequential execution, and no early downstream execution rules. :contentReference[oaicite:0]{index=0}

---

## Core rule

Artifact first.

Signal second.

Parent issue visibility required.

Correct order:

```text
1. Full artifact is posted on the parent issue.
2. Completion signal is posted on the parent issue.
3. Porky validates artifact + signal.
4. Porky advances the workflow.
```

Invalid order:

```text
1. Signal is emitted.
2. Artifact is missing.
3. Downstream phase starts.
```

The invalid order must block.

---

## Parent issue source of truth

The original parent issue is the canonical workflow record.

Signals may also be copied to subtasks for traceability, but final workflow state must be visible on the parent issue.

Subtask-only signals are not enough for final completion.

If a valid signal exists only on a subtask:

- Porky may use it for reconciliation
- the signal should be copied exactly to the parent issue
- final completion must wait until parent issue visibility is satisfied

Do not modify the signal when copying it.

---

## Canonical signal format

All signals should use this multiline format:

```text
SIGNAL:
type: <SIGNAL_TYPE>
producer: <AGENT_NAME>
issue: <PARENT_ISSUE_ID>
subtask_issue: <SUBTASK_ISSUE_ID>
status: <COMPLETE | FAILED>
artifact: <ARTIFACT_NAME>
timestamp: <TIMESTAMP>
```

Do not use inline signals.

Do not omit required fields.

Do not invent signal types.

---

## Valid status values

Allowed values:

```text
COMPLETE
FAILED
```

Invalid values:

```text
DONE
SUCCESS
FINISHED
OK
BLOCKED
```

Use `FAILED` for blocked states.

Use `COMPLETE` for successful phase completion.

---

## Valid completion signals

### AUDIT_READY

Produced by Babe.

Used when the full `audit` artifact has been published on the parent issue and is ready for Porky validation.

Required artifact:

```text
ARTIFACT:
type: audit
```

Required producer:

```text
Babe
```

Valid signal:

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: PIG-101
subtask_issue: PIG-102
status: COMPLETE
artifact: audit
timestamp: 2026-05-01T14:01:00Z
```

Important:

`AUDIT_READY` is valid only if the audit artifact includes all required sections, including:

```text
critical_flags
```

If no critical flags exist, the audit artifact must still include:

```text
critical_flags: []
```

---

### SYNTHESIS_READY

Produced by Porky.

Used when the full `strategy` artifact has been published on the parent issue and is ready for Hamm.

Required artifact:

```text
ARTIFACT:
type: strategy
```

Required producer:

```text
Porky
```

Valid signal:

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
issue: PIG-101
subtask_issue: PIG-101
status: COMPLETE
artifact: strategy
timestamp: 2026-05-01T14:10:00Z
```

Note:

If Porky emits this directly on the parent issue, `subtask_issue` may match the parent issue or be omitted depending on the workflow system.

When available, include it.

Important:

If Babe’s audit includes publishing risks, the strategy artifact must include:

```text
excluded_claims
safe_framing
```

`SYNTHESIS_READY` is invalid if the strategy ignores unresolved critical risks.

---

### CONTENT_SET_READY

Produced by Hamm.

Used when the full `content_set` artifact has been published on the parent issue and is ready for Porky validation and Pumba persistence.

Required artifact:

```text
ARTIFACT:
type: content_set
```

Required producer:

```text
Hamm
```

Valid signal:

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

Important:

`CONTENT_SET_READY` is valid only if:

- the full `content_set` artifact exists
- every content item has required metadata
- posts include `body`
- threads include `thread_id` and `tweets`
- excluded claims are avoided
- safe framing is used when required
- artifact is posted on the parent issue

---

### NOTION_SYNC_COMPLETE

Produced by Pumba.

Used when both Notion persistence outputs are complete and verified:

1. Content Pipeline entries
2. Research Babe page

Required artifact:

```text
ARTIFACT:
type: notion_content_pipeline
```

Required producer:

```text
Pumba
```

Valid signal:

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: notion_content_pipeline
timestamp: 2026-05-01T14:35:00Z
```

Required meaning:

```text
Content Pipeline persisted AND Research Babe persisted
```

This signal is invalid if Research Babe is missing.

This signal is invalid if content is stored in `Notes`.

This signal is invalid if the `notion_content_pipeline` artifact is missing.

---

### DELIVERY_COMPLETE

Produced by Pumba.

Used when the Telegram delivery summary has been sent and verified.

Required artifact:

```text
ARTIFACT:
type: telegram_delivery
```

Required producer:

```text
Pumba
```

Valid signal:

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: telegram_delivery
timestamp: 2026-05-01T14:40:00Z
```

Important:

`DELIVERY_COMPLETE` is valid only if:

- Telegram summary was sent once
- duplicate delivery check passed
- summary references Notion Content Pipeline
- summary references Research Babe
- `telegram_delivery` artifact exists on the parent issue

---

## Valid blocked signals

A `BLOCKED` signal means the workflow cannot safely continue.

Blocked signals use:

```text
status: FAILED
```

A `BLOCKED` signal does not mean the workflow is permanently failed.

It means the workflow needs correction before it can continue.

---

### BLOCKED by Babe

Used when Babe cannot produce a valid audit artifact.

```text
SIGNAL:
type: BLOCKED
producer: Babe
issue: PIG-101
subtask_issue: PIG-102
status: FAILED
artifact: audit
timestamp: 2026-05-01T14:00:00Z
```

Example reasons:

```text
Babe is blocked because parent_issue_id or subtask_issue_id is missing.
```

```text
Babe is blocked because audit.digital_presence failed and no usable audit can be produced.
```

```text
Babe is blocked because required research inputs are missing.
```

---

### BLOCKED by Hamm

Used when Hamm cannot produce a valid content set.

```text
SIGNAL:
type: BLOCKED
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: FAILED
artifact: content_generation
timestamp: 2026-05-01T14:15:00Z
```

Example reasons:

```text
Hamm is blocked because no valid strategy artifact exists.
```

```text
Hamm is blocked because excluded_claims are unclear and the content would risk unsafe claims.
```

```text
Hamm is blocked because Porky has not explicitly delegated the Content phase.
```

---

### BLOCKED by Pumba

Used when Pumba cannot persist or deliver safely after explicit delegation.

```text
SIGNAL:
type: BLOCKED
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: FAILED
artifact: persistence_or_delivery
timestamp: 2026-05-01T14:30:00Z
```

Example reasons:

```text
Pumba is blocked because the audit artifact is missing, so Research Babe cannot be persisted.
```

```text
Pumba is blocked because the content_set artifact is missing.
```

```text
Pumba is blocked because Notion verification failed.
```

Important:

Pumba should not emit noisy `BLOCKED` signals for harmless early wake-ups before Porky explicitly delegates the Notion + Delivery phase.

If Pumba wakes early, it should exit cleanly or leave a waiting note.

---

### BLOCKED by Porky

Used when Porky detects an invalid workflow state.

```text
SIGNAL:
type: BLOCKED
producer: Porky
issue: PIG-101
subtask_issue: PIG-101
status: FAILED
artifact: workflow
timestamp: 2026-05-01T14:25:00Z
```

Example reasons:

```text
Porky blocked the workflow because CONTENT_SET_READY exists but the full content_set artifact is missing.
```

```text
Porky blocked the workflow because Hamm was created before strategy validation passed.
```

```text
Porky blocked the workflow because a subtask is title-only and lacks required context.
```

```text
Porky blocked the workflow because Babe’s audit artifact is missing critical_flags.
```

---

## Waiting note for early Pumba wake-up

This is not a signal.

Use a waiting note when Pumba wakes before explicit delegation.

```text
Status: WAITING

Reason:
Pumba is awaiting explicit delegation from Porky for the Notion + Delivery phase.
Required upstream inputs are not yet validated or not yet delegated.
```

This should not be treated as workflow failure.

Pumba must not emit:

```text
NOTION_SYNC_COMPLETE
DELIVERY_COMPLETE
```

before explicit delegation and validation of required inputs.

---

## Timeout and recovery signals

### TIMEOUT_DETECTED

Produced by Porky when a required signal or artifact does not appear within the expected workflow window.

```text
SIGNAL:
type: TIMEOUT_DETECTED
producer: Porky
issue: PIG-101
subtask_issue: PIG-101
status: FAILED
artifact: workflow
timestamp: 2026-05-01T15:00:00Z
```

---

### RECOVERY_IN_PROGRESS

Produced by Porky when recovery begins.

```text
SIGNAL:
type: RECOVERY_IN_PROGRESS
producer: Porky
issue: PIG-101
subtask_issue: PIG-101
status: FAILED
artifact: workflow
timestamp: 2026-05-01T15:05:00Z
```

---

### RECOVERY_COMPLETE

Produced by Porky when recovery finishes successfully.

```text
SIGNAL:
type: RECOVERY_COMPLETE
producer: Porky
issue: PIG-101
subtask_issue: PIG-101
status: COMPLETE
artifact: workflow
timestamp: 2026-05-01T15:10:00Z
```

---

## Signal-to-artifact mapping

| Signal | Producer | Required artifact or verified output |
|---|---|---|
| `AUDIT_READY` | Babe | `audit` |
| `SYNTHESIS_READY` | Porky | `strategy` |
| `CONTENT_SET_READY` | Hamm | `content_set` |
| `NOTION_SYNC_COMPLETE` | Pumba | `notion_content_pipeline` |
| `DELIVERY_COMPLETE` | Pumba | `telegram_delivery` |
| `BLOCKED` | Any agent | Phase-specific blocked artifact |
| `TIMEOUT_DETECTED` | Porky | `workflow` |
| `RECOVERY_IN_PROGRESS` | Porky | `workflow` |
| `RECOVERY_COMPLETE` | Porky | `workflow` |

---

## Producer ownership

| Signal | Valid producer |
|---|---|
| `AUDIT_READY` | Babe |
| `SYNTHESIS_READY` | Porky |
| `CONTENT_SET_READY` | Hamm |
| `NOTION_SYNC_COMPLETE` | Pumba |
| `DELIVERY_COMPLETE` | Pumba |
| `BLOCKED` | Any agent |
| `TIMEOUT_DETECTED` | Porky |
| `RECOVERY_IN_PROGRESS` | Porky |
| `RECOVERY_COMPLETE` | Porky |

Wrong-producer signals are invalid.

Examples:

- Pumba cannot emit `CONTENT_SET_READY`
- Hamm cannot emit `AUDIT_READY`
- Babe cannot emit `SYNTHESIS_READY`
- Porky cannot emit `DELIVERY_COMPLETE`

---

## Sequential execution requirements

Signals are only meaningful within the correct phase order.

Canonical order:

```text
AUDIT_READY → SYNTHESIS_READY → CONTENT_SET_READY → NOTION_SYNC_COMPLETE → DELIVERY_COMPLETE
```

Hamm should not emit `CONTENT_SET_READY` before `SYNTHESIS_READY`.

Pumba should not emit `NOTION_SYNC_COMPLETE` before `CONTENT_SET_READY`.

Pumba should not emit `DELIVERY_COMPLETE` before `NOTION_SYNC_COMPLETE`.

Porky may reconcile valid artifacts if a signal is missing, but downstream execution must still be safe and sequential.

---

## Parent issue propagation example

If Hamm emits this signal in subtask `PIG-103`:

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

Hamm must copy the exact same signal to the parent issue `PIG-101`.

Do not change:

- signal type
- producer
- issue
- subtask issue
- status
- artifact
- timestamp

The same rule applies to artifacts.

If the artifact exists only on the subtask, it must also be visible on the parent issue before final completion.

---

## Pumba artifact examples

### notion_content_pipeline artifact

Before emitting `NOTION_SYNC_COMPLETE`, Pumba must publish:

```text
ARTIFACT:
type: notion_content_pipeline
issue: PIG-101
subtask_issue: PIG-104
content_items_persisted: 3
expected_content_items: 3
research_babe_persisted: true
duplicates_found: false
notes_used_for_content: false
timestamp: 2026-05-01T14:35:00Z
verification:
  content_pipeline_entries_verified: true
  research_babe_page_verified: true
  body_content_verified: true
  required_properties_verified: true
  thread_blocks_verified: true
  duplicate_check_passed: true
```

Then Pumba may emit:

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: notion_content_pipeline
timestamp: 2026-05-01T14:35:00Z
```

---

### telegram_delivery artifact

Before emitting `DELIVERY_COMPLETE`, Pumba must publish:

```text
ARTIFACT:
type: telegram_delivery
issue: PIG-101
subtask_issue: PIG-104
telegram_summary_sent: true
duplicate_delivery: false
timestamp: 2026-05-01T14:40:00Z
summary_includes:
  parent_issue_id: true
  date_range: true
  total_content_items: true
  number_of_posts: true
  number_of_threads: true
  platforms_used: true
  languages_used: true
  notion_content_pipeline_status: true
  research_babe_status: true
  duplicate_check_result: true
verification:
  delivery_sent: true
  delivery_not_duplicated: true
  summary_content_verified: true
  notion_status_referenced: true
  research_babe_referenced: true
```

Then Pumba may emit:

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: telegram_delivery
timestamp: 2026-05-01T14:40:00Z
```

---

## Invalid signal examples

### Invalid: inline signal

```text
SIGNAL: type: CONTENT_SET_READY producer: Hamm status: COMPLETE
```

Why invalid:

- Inline format
- Missing issue
- Missing subtask issue
- Missing artifact
- Missing timestamp

---

### Invalid: missing artifact field

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
timestamp: 2026-05-01T14:20:00Z
```

Why invalid:

- Missing `artifact`

---

### Invalid: wrong producer

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

Why invalid:

- `CONTENT_SET_READY` must be produced by Hamm, not Pumba.

---

### Invalid: unsupported status

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: PIG-101
subtask_issue: PIG-102
status: DONE
artifact: audit
timestamp: 2026-05-01T14:01:00Z
```

Why invalid:

- `DONE` is not a valid status.
- Use `COMPLETE`.

---

### Invalid: unsupported signal type

```text
SIGNAL:
type: CONTENT_DONE
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

Why invalid:

- `CONTENT_DONE` is not an allowed signal type.
- Use `CONTENT_SET_READY`.

---

### Invalid: completion signal before artifact

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

Why invalid:

- This is invalid if the full `ARTIFACT: type: content_set` block is not visible before the signal.

---

### Invalid: subtask-only signal

Signal exists only on `PIG-103`:

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

But the parent issue `PIG-101` does not contain the same signal.

Why invalid for final completion:

- Parent issue is the source of truth.
- Porky may use the signal for reconciliation.
- Final workflow completion still requires parent issue visibility.

---

### Invalid: AUDIT_READY without critical_flags

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: PIG-101
subtask_issue: PIG-102
status: COMPLETE
artifact: audit
timestamp: 2026-05-01T14:01:00Z
```

Why invalid:

- This is invalid if the audit artifact does not include `critical_flags`.
- `critical_flags` must exist even if empty.

---

### Invalid: SYNTHESIS_READY while strategy ignores critical risk

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
issue: PIG-101
subtask_issue: PIG-101
status: COMPLETE
artifact: strategy
timestamp: 2026-05-01T14:10:00Z
```

Why invalid:

- This is invalid if Babe flagged a publishing risk and the strategy does not block, exclude, or safely frame it.
- If risks exist, the strategy should include `excluded_claims` and `safe_framing`.

---

### Invalid: CONTENT_SET_READY while content uses excluded claims

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-101
subtask_issue: PIG-103
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:20:00Z
```

Why invalid:

- This is invalid if the content uses a claim listed under `excluded_claims`.
- Hamm must respect Porky’s strategy constraints.

---

### Invalid: NOTION_SYNC_COMPLETE without Research Babe

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: notion_content_pipeline
timestamp: 2026-05-01T14:35:00Z
```

Why invalid:

- This is invalid if Content Pipeline entries exist but the Research Babe page is missing.

`NOTION_SYNC_COMPLETE` means both content persistence and Research Babe persistence are complete.

---

### Invalid: NOTION_SYNC_COMPLETE without notion_content_pipeline artifact

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: notion_content_pipeline
timestamp: 2026-05-01T14:35:00Z
```

Why invalid:

- The signal is invalid if the `ARTIFACT: type: notion_content_pipeline` block is missing.
- Porky needs the artifact to validate persistence.

---

### Invalid: DELIVERY_COMPLETE without telegram_delivery artifact

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: COMPLETE
artifact: telegram_delivery
timestamp: 2026-05-01T14:40:00Z
```

Why invalid:

- The signal is invalid if the `ARTIFACT: type: telegram_delivery` block is missing.
- Porky needs the artifact to validate delivery.

---

### Invalid: Pumba BLOCKED before explicit delegation

```text
SIGNAL:
type: BLOCKED
producer: Pumba
issue: PIG-101
subtask_issue: PIG-104
status: FAILED
artifact: persistence_or_delivery
timestamp: 2026-05-01T14:05:00Z
```

Why potentially invalid:

- If Porky has not explicitly delegated the Notion + Delivery phase yet, this is a noisy early blocked signal.
- Pumba should wait or exit cleanly instead.
- Missing upstream artifacts are expected before Pumba is delegated.

---

## Best practices

- Publish the artifact before emitting the signal.
- Post completion artifacts and signals on the parent issue.
- Copy artifacts and signals to subtasks only for traceability.
- Use the exact canonical signal format.
- Use exact signal names.
- Use exact artifact names.
- Include parent issue ID.
- Include subtask issue ID when available.
- Emit one completion signal per completed phase.
- Use `BLOCKED` instead of guessing after explicit delegation.
- Use a waiting note, not `BLOCKED`, for harmless early Pumba wake-up.
- Never emit completion signals for partial work.
- Never emit signals for another agent’s responsibility.
- Never treat task status as a replacement for signals and artifacts.
- Never treat subtask-only evidence as enough for final completion.

---

## Summary

Signals are the workflow transition markers of Open Agentic CMO.

They make agent communication explicit, structured, and auditable.

A signal is valid only when it is:

- correctly formatted
- produced by the correct agent
- tied to the correct parent issue
- mapped to the correct artifact
- visible on the parent issue
- backed by a valid artifact or verified output
- consistent with the current workflow phase

Signals move the workflow forward only when Porky can validate the underlying artifact.

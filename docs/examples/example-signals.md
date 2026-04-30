# Example Signals

This example shows valid and invalid signals used by Open Agentic CMO.

Signals are explicit workflow transition markers.

A signal tells Porky that a workflow phase has reached a defined state.

However, a signal alone is not enough.

Every completion signal must map to a valid artifact.

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

## Valid completion signals

### AUDIT_READY

Produced by Babe.

Used when the full `audit` artifact has been published and is ready for Porky validation.

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

Required artifact:

```text
ARTIFACT:
type: audit
```

---

### SYNTHESIS_READY

Produced by Porky.

Used when the full `strategy` artifact has been published and is ready for Hamm.

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

Required artifact:

```text
ARTIFACT:
type: strategy
```

Note:

If Porky emits this directly on the parent issue, `subtask_issue` may match the parent issue or be omitted depending on the workflow system. When available, include it.

---

### CONTENT_SET_READY

Produced by Hamm.

Used when the full `content_set` artifact has been published and is ready for Porky validation and Pumba persistence.

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

Required artifact:

```text
ARTIFACT:
type: content_set
```

---

### NOTION_SYNC_COMPLETE

Produced by Pumba.

Used when both Notion persistence outputs are complete and verified:

1. Content Pipeline entries
2. Research Babe page

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

---

### DELIVERY_COMPLETE

Produced by Pumba.

Used when the Telegram delivery summary has been sent and verified.

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

Required artifact or verified output:

```text
ARTIFACT:
type: telegram_delivery
```

---

## Valid blocked signals

A `BLOCKED` signal means the workflow cannot safely continue.

Blocked signals use:

```text
status: FAILED
```

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

Example reason:

```text
Babe is blocked because parent_issue_id or subtask_issue_id is missing.
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

Example reason:

```text
Hamm is blocked because no valid strategy artifact exists.
```

---

### BLOCKED by Pumba

Used when Pumba cannot persist or deliver safely.

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

Example reason:

```text
Pumba is blocked because the audit artifact is missing, so Research Babe cannot be persisted.
```

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

Example reason:

```text
Porky blocked the workflow because CONTENT_SET_READY exists but the full content_set artifact is missing.
```

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

## Best practices

- Publish the artifact before emitting the signal.
- Use the exact canonical signal format.
- Use exact signal names.
- Use exact artifact names.
- Include parent issue ID.
- Include subtask issue ID when available.
- Propagate subtask signals to the parent issue.
- Emit one completion signal per completed phase.
- Use `BLOCKED` instead of guessing.
- Never emit completion signals for partial work.
- Never emit signals for another agent’s responsibility.

---

## Summary

Signals are the workflow transition markers of Open Agentic CMO.

They make agent communication explicit, structured, and auditable.

A signal is valid only when it is:

- correctly formatted
- produced by the correct agent
- tied to the correct issue
- mapped to the correct artifact
- backed by a valid artifact or verified output

Signals move the workflow forward only when Porky can validate the underlying artifact.

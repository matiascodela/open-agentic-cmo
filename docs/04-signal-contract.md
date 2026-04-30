# Signal Contract

Signals are the explicit workflow transition markers used by Open Agentic CMO.

They tell Porky that a specific phase has reached a defined state, such as:

- Research is ready
- Strategy is ready
- Content is ready
- Notion persistence is complete
- Delivery is complete
- A phase is blocked

Signals are required for deterministic orchestration.

However, a signal alone is never enough.

Every completion signal must correspond to a valid artifact.

---

## Why signals exist

AI agents often fail when they communicate completion using informal text.

Examples of weak completion messages:

- “Done”
- “Looks good”
- “Content is ready”
- “Research completed”
- “Everything is available in the comments”
- “Notion was updated”

These messages are not reliable because they are not structured, not machine-checkable, and often do not prove that the required output exists.

Open Agentic CMO uses signals to make workflow transitions explicit and verifiable.

---

## Core principle

A signal declares state.

An artifact proves work.

This means:

- A signal without an artifact is incomplete.
- An artifact without a signal may be reconciled.
- A malformed signal is invalid.
- A signal with missing fields is invalid.
- A signal from the wrong producer is invalid.
- A completion signal should never be emitted before the artifact exists.

---

## Canonical signal format

All signals must use this multiline format:

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

Invalid:

```text
SIGNAL: type: CONTENT_SET_READY producer: Hamm status: COMPLETE
```

Valid:

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-77
subtask_issue: PIG-79
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:22:00Z
```

---

## Required fields

Every signal must include:

| Field | Required | Description |
|---|---:|---|
| `type` | Yes | Signal type |
| `producer` | Yes | Agent that emitted the signal |
| `issue` | Yes | Parent issue ID |
| `subtask_issue` | Preferred | Subtask issue ID when available |
| `status` | Yes | Completion or failure state |
| `artifact` | Yes | Artifact associated with the signal |
| `timestamp` | Yes | Time the signal was emitted |

Although `subtask_issue` may be unavailable in some edge cases, it should be included whenever the signal originates from a subtask.

---

## Valid status values

Allowed status values:

- `COMPLETE`
- `FAILED`

Do not invent alternative status values.

Invalid examples:

- `DONE`
- `SUCCESS`
- `ERROR`
- `READY`
- `PENDING`
- `IN_PROGRESS`

---

## Allowed signal types

Open Agentic CMO currently supports the following signal types:

| Signal | Producer | Meaning |
|---|---|---|
| `AUDIT_READY` | Babe | Audit artifact is complete |
| `SYNTHESIS_READY` | Porky | Strategy artifact is complete |
| `CONTENT_SET_READY` | Hamm | Content set artifact is complete |
| `NOTION_SYNC_COMPLETE` | Pumba | Notion persistence is complete |
| `DELIVERY_COMPLETE` | Pumba | Telegram delivery is complete |
| `BLOCKED` | Any agent | Phase cannot continue |
| `RETRY_REQUIRED` | Porky or specialist agent | Retry is required |
| `TIMEOUT_DETECTED` | Porky | Workflow stalled waiting for a signal or artifact |
| `RECOVERY_IN_PROGRESS` | Porky | Recovery process has started |
| `RECOVERY_COMPLETE` | Porky | Recovery process has completed |

Any signal type not listed here is invalid.

---

## Producer ownership

Each signal has an expected producer.

### Babe

Babe may emit:

- `AUDIT_READY`
- `BLOCKED`

Babe must not emit:

- `SYNTHESIS_READY`
- `CONTENT_SET_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

---

### Porky

Porky may emit:

- `SYNTHESIS_READY`
- `BLOCKED`
- `TIMEOUT_DETECTED`
- `RETRY_REQUIRED`
- `RECOVERY_IN_PROGRESS`
- `RECOVERY_COMPLETE`

Porky normally does not emit:

- `AUDIT_READY`
- `CONTENT_SET_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

---

### Hamm

Hamm may emit:

- `CONTENT_SET_READY`
- `BLOCKED`

Hamm must not emit:

- `AUDIT_READY`
- `SYNTHESIS_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

---

### Pumba

Pumba may emit:

- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`
- `BLOCKED`

Pumba must not emit:

- `AUDIT_READY`
- `SYNTHESIS_READY`
- `CONTENT_SET_READY`

---

## Signal-to-artifact mapping

Completion signals must map to artifacts.

| Signal | Required artifact |
|---|---|
| `AUDIT_READY` | `audit` |
| `SYNTHESIS_READY` | `strategy` |
| `CONTENT_SET_READY` | `content_set` |
| `NOTION_SYNC_COMPLETE` | `notion_content_pipeline` |
| `DELIVERY_COMPLETE` | `telegram_delivery` |

A signal is incomplete if the required artifact does not exist.

---

## Completion signals

Completion signals use `status: COMPLETE`.

Example:

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: PIG-77
subtask_issue: PIG-78
status: COMPLETE
artifact: audit
timestamp: 2026-05-01T14:00:00Z
```

Completion signals should only be emitted after the corresponding artifact is visible and valid.

---

## Blocked signals

A `BLOCKED` signal means an agent cannot continue safely.

Blocked signals use:

```text
status: FAILED
```

Example:

```text
SIGNAL:
type: BLOCKED
producer: Hamm
issue: PIG-77
subtask_issue: PIG-79
status: FAILED
artifact: content_generation
timestamp: 2026-05-01T14:10:00Z
```

A blocked message should also explain:

- blocked phase
- responsible agent
- missing or invalid requirement
- required next action

---

## Timeout signals

Porky may emit `TIMEOUT_DETECTED` when the workflow stalls.

Example:

```text
SIGNAL:
type: TIMEOUT_DETECTED
producer: Porky
issue: PIG-77
status: FAILED
artifact: workflow
timestamp: 2026-05-01T15:00:00Z
```

After detecting timeout, Porky should attempt reconciliation by scanning parent issue and subtask comments for valid artifacts.

---

## Recovery signals

Recovery signals are used when Porky attempts to restore workflow progress.

Example:

```text
SIGNAL:
type: RECOVERY_IN_PROGRESS
producer: Porky
issue: PIG-77
status: FAILED
artifact: workflow
timestamp: 2026-05-01T15:05:00Z
```

Example:

```text
SIGNAL:
type: RECOVERY_COMPLETE
producer: Porky
issue: PIG-77
status: COMPLETE
artifact: workflow
timestamp: 2026-05-01T15:10:00Z
```

Recovery should only complete when valid artifacts or corrected signals allow safe continuation.

---

## Parent issue propagation

Signals emitted in subtasks must also be posted in the parent issue.

The signal must be copied exactly.

Do not:

- regenerate the signal
- change the issue ID
- change the subtask issue ID
- change the timestamp
- alter the artifact name
- alter the status

This ensures Porky can scan both parent issue and subtasks reliably.

---

## Signal normalization

Signals may originate from:

- parent issue comments
- subtask comments

Porky normalizes valid subtask signals to the parent workflow.

This means a valid signal emitted in a subtask can still advance the parent workflow if:

- the subtask belongs to the workflow
- the signal is well-formed
- the artifact exists
- the artifact passes validation

---

## Signal deduplication

Duplicate signals count as one logical signal.

If the same valid signal appears in both the subtask and the parent issue, Porky treats it as a single workflow event.

If duplicate signals conflict, Porky should prefer the most complete valid signal and log the conflict.

---

## Invalid signals

A signal is invalid if:

- it is inline instead of multiline
- it is missing required fields
- it uses an unsupported signal type
- it uses an unsupported status
- it has the wrong producer
- it references the wrong parent issue
- it references an unrelated subtask
- it points to an artifact that does not exist
- it is emitted before the artifact exists
- it uses a non-canonical artifact name

---

## Examples of invalid signals

### Inline signal

```text
SIGNAL: type: CONTENT_SET_READY producer: Hamm status: COMPLETE
```

Invalid because the signal is compressed and missing required fields.

---

### Missing artifact

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-77
subtask_issue: PIG-79
status: COMPLETE
timestamp: 2026-05-01T14:22:00Z
```

Invalid because `artifact` is missing.

---

### Wrong producer

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Pumba
issue: PIG-77
subtask_issue: PIG-80
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:22:00Z
```

Invalid because `CONTENT_SET_READY` must be produced by Hamm.

---

### Signal without artifact

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: PIG-77
subtask_issue: PIG-79
status: COMPLETE
artifact: content_set
timestamp: 2026-05-01T14:22:00Z
```

This is invalid if the full `content_set` artifact is not visible and valid.

---

## Best practices

Use one completion signal per completed phase.

Post the artifact before the signal.

Use exact signal names.

Use exact artifact names.

Propagate the same signal to the parent issue.

Do not emit completion signals for partial work.

Do not emit multiple competing completion signals.

Use `BLOCKED` instead of guessing or inventing missing data.

---

## Summary

Signals are the workflow transition layer of Open Agentic CMO.

They provide structure, determinism, and traceability.

But signals are not proof by themselves.

A valid transition requires:

1. A valid signal
2. A valid artifact
3. A passing validation gate

This is what allows Open Agentic CMO to operate as a reliable multi-agent workflow instead of a loose chain of text responses.

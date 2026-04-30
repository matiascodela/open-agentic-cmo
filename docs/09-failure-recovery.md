# Failure Recovery

Failure recovery defines how Open Agentic CMO handles incomplete, invalid, or corrupted workflow states.

The goal is not to avoid every possible failure.

The goal is to make failures explicit, recoverable, and safe.

Open Agentic CMO should never silently continue when a required signal, artifact, or persisted output is missing or invalid.

---

## Core principle

A blocked workflow is safer than a corrupted workflow.

When the system cannot prove that a phase completed correctly, it must block.

The system should only continue when:

1. The required signal exists or can be reconciled.
2. The required artifact exists.
3. The artifact passes validation.
4. Downstream execution will not duplicate or corrupt data.

---

## Recovery ownership

Failure recovery is primarily owned by Porky.

Porky is responsible for:

- Scanning parent issue comments
- Scanning subtask comments
- Extracting signals
- Extracting artifacts
- Validating signals
- Validating artifacts
- Detecting missing or invalid outputs
- Blocking unsafe transitions
- Requesting corrections from the responsible agent
- Resuming the workflow when corrected artifacts exist

Specialist agents are responsible for fixing their own outputs.

| Failure area | Responsible agent |
|---|---|
| Missing or invalid audit artifact | Babe |
| Missing or invalid strategy artifact | Porky |
| Missing or invalid content_set artifact | Hamm |
| Notion persistence issue | Pumba |
| Missing Research Babe page | Pumba |
| Telegram delivery issue | Pumba |

---

## General recovery process

When Porky detects a failure:

1. Identify the current phase.
2. Identify the missing or invalid requirement.
3. Identify the responsible agent.
4. Block the workflow.
5. Write a clear reason.
6. Request the exact correction needed.
7. Do not advance downstream.
8. Resume only after the corrected artifact or output passes validation.

Porky should not restart the full workflow if only one phase failed.

---

## Blocking message format

A blocking message should include:

- Blocked phase
- Responsible agent
- Problem detected
- Why it is unsafe to continue
- Required next action

Example:

```text
Blocked phase: Content
Responsible agent: Hamm
Problem: CONTENT_SET_READY was emitted, but no full content_set artifact exists.
Why blocked: Pumba cannot persist content without validated content_items.
Required next action: Hamm must publish the complete ARTIFACT: type: content_set block before emitting CONTENT_SET_READY again.
```

---

## BLOCKED signal

When blocking the workflow, the responsible agent or Porky should emit a `BLOCKED` signal.

Example:

```text
SIGNAL:
type: BLOCKED
producer: Porky
issue: <PARENT_ISSUE_ID>
status: FAILED
artifact: workflow
timestamp: <TIMESTAMP>
```

Specialist agent example:

```text
SIGNAL:
type: BLOCKED
producer: Hamm
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
status: FAILED
artifact: content_generation
timestamp: <TIMESTAMP>
```

A `BLOCKED` signal does not mean the workflow is permanently failed.

It means the workflow cannot safely continue until the issue is corrected.

---

## Failure type 1 — Signal exists but artifact is missing

### Example

Hamm emits:

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

But no full `ARTIFACT: type: content_set` block exists.

### Expected behavior

Porky must:

1. Treat the signal as incomplete.
2. Block the workflow.
3. Prevent Pumba from running.
4. Request the full artifact from Hamm.

### Recovery action

Hamm must publish the complete content set artifact.

Required format:

```text
ARTIFACT:
type: content_set
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
item_count: <NUMBER>
content_items:
...
```

Only after the artifact exists and passes validation may Porky continue.

---

## Failure type 2 — Artifact exists but signal is missing

### Example

Babe publishes a valid `audit` artifact but forgets to emit `AUDIT_READY`.

### Expected behavior

Porky may reconcile the workflow.

Porky should:

1. Validate the artifact.
2. Confirm it belongs to the parent workflow.
3. Register internal completion for the phase.
4. Request signal propagation from Babe if needed.
5. Continue safely if the artifact is valid.

### Recovery action

Babe should still post the missing signal:

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: audit
timestamp: <TIMESTAMP>
```

The workflow may continue once Porky has validated the artifact.

---

## Failure type 3 — Artifact exists but is incomplete

### Example

Babe posts an audit artifact without uncertainty or recommended content angles.

### Expected behavior

Porky must block.

The audit artifact is incomplete because required sections are missing.

### Recovery action

Babe must update the audit artifact with all required sections:

- Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles

Porky should not synthesize strategy until the audit artifact passes validation.

---

## Failure type 4 — Strategy artifact is missing or too vague

### Example

Porky posts a strategy summary but does not publish a structured `strategy` artifact.

Or the strategy says:

```text
Create engaging content about financial education.
```

### Expected behavior

Porky must not delegate to Hamm.

The strategy must be structured enough for Hamm to generate content without interpretation.

### Recovery action

Porky must publish a complete strategy artifact with:

- Date range
- Expected items
- Messaging pillars
- Content angles
- Weekly structure
- Brand constraints
- Content rules

Only then should Porky emit `SYNTHESIS_READY`.

---

## Failure type 5 — Content set is incomplete

### Example

Hamm publishes only a summary:

```text
Generated 8 content items:
- 4 X threads
- 2 X posts
- 2 LinkedIn posts
```

But does not publish the full content.

### Expected behavior

Porky must block.

A summary is not a content artifact.

### Recovery action

Hamm must publish the full `content_set` artifact with every item included.

Each item must include:

- Title
- Platform
- Language
- Content type
- Scheduled date
- Vertical
- Version
- Week
- Tags and mentions

Posts must include:

- Body

Threads must include:

- Thread ID
- Tweets array

---

## Failure type 6 — Content item missing required fields

### Example

A content item is missing:

- `Version`
- `Week`
- `Scheduled Date`
- `Content Type`

### Expected behavior

Porky must reject the `content_set`.

Pumba must not persist incomplete content.

### Recovery action

Hamm must update the `content_set` artifact and fill all required fields.

If the content was already persisted incorrectly, Pumba must update the Notion pages after the corrected artifact is available.

---

## Failure type 7 — Thread structure is invalid

### Example

A thread contains:

- only 2 tweets
- duplicated tweets
- unordered tweets
- one merged block instead of an array

### Expected behavior

Porky must reject the `content_set`.

Pumba must not persist invalid thread content.

### Recovery action

Hamm must correct the thread so it includes:

- 4–7 tweets
- ordered tweet array
- no duplicate tweets
- logical progression

---

## Failure type 8 — Content persisted into Notes

### Example

Pumba creates Notion pages but stores post or thread content inside `Notes`.

### Expected behavior

Porky must reject `NOTION_SYNC_COMPLETE`.

Content stored in `Notes` is considered invalid persistence.

### Recovery action

Pumba must update the Notion pages:

- Move post body into the page body
- Split thread tweets into ordered page body blocks
- Remove content from `Notes` if safe
- Preserve required properties

After correction, Pumba may emit `NOTION_SYNC_COMPLETE`.

---

## Failure type 9 — Required Notion properties are missing

### Example

A Notion content page exists but one or more required properties are empty:

- Version
- Week
- Content Type
- Scheduled Date
- Platform
- Vertical

### Expected behavior

Porky must reject `NOTION_SYNC_COMPLETE`.

### Recovery action

Pumba must update the existing Notion page with the correct properties from the `content_set` artifact.

Pumba must not create a duplicate page.

---

## Failure type 10 — Research Babe page is missing

### Example

Pumba persists all content items but does not create or update the Research Babe page.

### Expected behavior

Porky must reject `NOTION_SYNC_COMPLETE`.

`NOTION_SYNC_COMPLETE` means both:

1. Content Pipeline entries were persisted.
2. Research Babe page was persisted.

Anything less is incomplete.

### Recovery action

Pumba must create or update:

```text
Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>
```

The page must include:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Source / Workflow Metadata

---

## Failure type 11 — Duplicate Notion entries

### Example

The same content item appears twice in the Notion Content Pipeline.

### Expected behavior

Porky must reject Notion validation.

### Recovery action

Pumba must deduplicate safely.

Use the content deduplication key:

```text
parent_issue_id + scheduled_date + platform + title
```

Pumba should:

1. Identify duplicate entries.
2. Preserve the correct entry.
3. Remove, archive, or mark duplicate entries according to the workspace rules.
4. Verify no duplicates remain.
5. Avoid rewriting valid content unnecessarily.

---

## Failure type 12 — Duplicate Research Babe pages

### Example

Two Research Babe pages exist for the same parent issue.

### Expected behavior

Porky must reject Notion validation.

### Recovery action

Pumba must deduplicate safely.

Use the research deduplication key:

```text
parent_issue_id + babe_research
```

Pumba should preserve the correct page and remove, archive, or mark duplicates according to the workspace rules.

---

## Failure type 13 — Telegram summary missing

### Example

Notion persistence is complete, but no Telegram summary was sent.

### Expected behavior

Porky must not complete the workflow.

### Recovery action

Pumba must send the Telegram summary and emit:

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: telegram_delivery
timestamp: <TIMESTAMP>
```

---

## Failure type 14 — Duplicate Telegram summary

### Example

Pumba sends the same Telegram summary twice.

### Expected behavior

Porky should identify duplicate delivery and block final completion if duplicate delivery violates the workflow rules.

### Recovery action

Pumba should not resend valid summaries.

If a duplicate was already sent, Pumba should record that duplicate delivery occurred and avoid further sends.

---

## Failure type 15 — Downstream agent executed too early

### Example

Pumba runs before Hamm has produced a valid `content_set`.

### Expected behavior

Pumba should block.

Porky should prevent future early execution by enforcing downstream execution rules.

### Recovery action

Porky must wait for:

- valid audit artifact
- valid strategy artifact
- valid content_set artifact

before allowing Pumba to run.

---

## Failure type 16 — Wrong producer emits a signal

### Example

Pumba emits `CONTENT_SET_READY`.

### Expected behavior

Porky must treat the signal as invalid.

`CONTENT_SET_READY` can only be emitted by Hamm.

### Recovery action

The correct agent must emit the correct signal after producing the correct artifact.

---

## Failure type 17 — Malformed signal

### Example

```text
SIGNAL: type: CONTENT_SET_READY producer: Hamm status: COMPLETE
```

### Expected behavior

Porky must treat the signal as invalid.

Signals must use the canonical multiline format.

### Recovery action

The producing agent must repost the signal correctly.

---

## Failure type 18 — Parent issue propagation missing

### Example

Hamm emits `CONTENT_SET_READY` in the subtask but does not propagate the same signal to the parent issue.

### Expected behavior

Porky may still find the subtask signal if it scans subtasks.

However, the workflow is not fully compliant.

### Recovery action

Hamm must copy the exact same signal block to the parent issue.

The signal must not be regenerated or modified.

---

## Failure type 19 — Timeout or stalled workflow

### Example

Porky is waiting for `CONTENT_SET_READY`, but no signal or artifact appears.

### Expected behavior

Porky may emit:

```text
SIGNAL:
type: TIMEOUT_DETECTED
producer: Porky
issue: <PARENT_ISSUE_ID>
status: FAILED
artifact: workflow
timestamp: <TIMESTAMP>
```

Then Porky should attempt reconciliation.

### Recovery action

Porky should:

1. Scan parent issue.
2. Scan all subtasks.
3. Look for valid artifacts.
4. If valid artifact exists, continue.
5. If no valid artifact exists, block and request the missing output.

---

## Failure type 20 — Partial Notion persistence

### Example

Pumba persisted 2 of 3 content items.

### Expected behavior

Porky must reject `NOTION_SYNC_COMPLETE`.

### Recovery action

Pumba must resume only the missing persistence steps.

Pumba must not recreate pages that already exist and are valid.

---

## Recovery checklist

When recovering a workflow, verify:

- Which phase is blocked?
- Which agent owns the missing or invalid output?
- Does the required artifact exist?
- Does the required signal exist?
- Is the signal valid?
- Is the artifact valid?
- Has the signal been propagated to the parent issue?
- Has downstream execution already occurred?
- Would retrying create duplicates?
- Can the system resume from the current state?
- Is Notion already partially updated?
- Is Telegram already sent?

---

## Safe retry rules

Retries must be idempotent.

Before retrying, check:

- Does a valid artifact already exist?
- Does a valid Notion page already exist?
- Does a valid Research Babe page already exist?
- Was Telegram already sent?
- Is the missing work limited to one phase?

Retries should resume from the failed phase, not from the beginning.

---

## What not to do during recovery

Do not:

- Restart the full workflow unnecessarily
- Regenerate valid research
- Regenerate valid content
- Create duplicate Notion pages
- Resend Telegram summaries
- Ignore malformed signals
- Accept summaries as artifacts
- Invent missing fields
- Move forward with corrupted state

---

## Recovery examples

### Example 1 — Missing content artifact

Problem:

Hamm emitted `CONTENT_SET_READY`, but no full content artifact exists.

Resolution:

- Porky blocks.
- Hamm publishes full `content_set`.
- Porky validates.
- Pumba runs.

---

### Example 2 — Content in Notes

Problem:

Pumba persisted content into `Notes`.

Resolution:

- Porky rejects `NOTION_SYNC_COMPLETE`.
- Pumba moves content into page body.
- Pumba verifies Notion state.
- Pumba re-emits `NOTION_SYNC_COMPLETE`.

---

### Example 3 — Missing Research Babe

Problem:

Content pages exist, but Research Babe is missing.

Resolution:

- Porky rejects `NOTION_SYNC_COMPLETE`.
- Pumba creates Research Babe page.
- Pumba verifies both content and research persistence.
- Pumba emits `NOTION_SYNC_COMPLETE`.

---

## Summary

Failure recovery in Open Agentic CMO is based on a simple rule:

If the system cannot prove that the workflow state is correct, it must block.

Recovery should be:

- explicit
- phase-specific
- idempotent
- artifact-driven
- safe for downstream execution

The system should never hide failure behind a successful-looking signal.

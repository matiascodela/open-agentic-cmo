# Failure Recovery

Failure recovery defines how Open Agentic CMO handles incomplete, invalid, premature, or corrupted workflow states.

The goal is not to avoid every possible failure.

The goal is to make failures explicit, recoverable, and safe.

Open Agentic CMO should never silently continue when a required signal, artifact, subtask description, or persisted output is missing or invalid.

---

## Core principle

A blocked workflow is safer than a corrupted workflow.

When the system cannot prove that a phase completed correctly, it must block.

The system should only continue when:

1. The required signal exists or can be reconciled.
2. The required artifact exists.
3. The artifact passes validation.
4. The responsible agent is the correct producer.
5. The artifact and signal are visible on the parent issue.
6. Downstream execution will not duplicate or corrupt data.
7. The next delegated subtask has a complete description.

---

## Canonical workflow order

Failure recovery must preserve the canonical workflow order:

```text
Porky → Babe → Porky → Hamm → Porky → Pumba → Porky
```

The workflow must not recover by skipping phases or allowing downstream agents to run early.

Required order:

1. Porky delegates Babe.
2. Babe produces `audit`.
3. Porky validates `audit`.
4. Porky produces `strategy`.
5. Porky delegates Hamm.
6. Hamm produces `content_set`.
7. Porky validates `content_set`.
8. Porky delegates Pumba.
9. Pumba persists Notion + Research Babe.
10. Pumba sends Telegram.
11. Porky validates final state.

If recovery would violate this order, it must block.

---

## Parent issue as source of truth

The original parent issue is the canonical workflow record.

Subtasks may contain useful execution details, but final workflow state must be visible on the parent issue.

Required parent issue evidence:

- `ARTIFACT: type: audit`
- `SIGNAL: type: AUDIT_READY`
- `ARTIFACT: type: strategy`
- `SIGNAL: type: SYNTHESIS_READY`
- `ARTIFACT: type: content_set`
- `SIGNAL: type: CONTENT_SET_READY`
- `ARTIFACT: type: notion_content_pipeline`
- `SIGNAL: type: NOTION_SYNC_COMPLETE`
- `ARTIFACT: type: telegram_delivery`
- `SIGNAL: type: DELIVERY_COMPLETE`

If an artifact or signal exists only in a subtask, Porky may use it for reconciliation, but final completion requires parent issue visibility.

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
- Validating parent issue visibility
- Validating subtask description completeness
- Detecting premature downstream execution
- Detecting missing or invalid outputs
- Blocking unsafe transitions
- Requesting corrections from the responsible agent
- Resuming the workflow when corrected artifacts exist

Specialist agents are responsible for fixing their own outputs.

| Failure area | Responsible agent |
|---|---|
| Missing or invalid audit artifact | Babe |
| Missing `critical_flags` | Babe |
| Babe unbounded reference discovery | Babe |
| Missing or invalid strategy artifact | Porky |
| Missing `excluded_claims` / `safe_framing` when risks exist | Porky |
| Title-only or incomplete subtask | Porky |
| Missing or invalid `content_set` artifact | Hamm |
| Unsafe excluded claim used in content | Hamm |
| Notion persistence issue | Pumba |
| Missing Research Babe page | Pumba |
| Telegram delivery issue | Pumba |
| Early Pumba execution before delegation | Pumba + Porky |
| Early Hamm execution before strategy | Hamm + Porky |

---

## General recovery process

When Porky detects a failure:

1. Identify the current phase.
2. Identify the missing or invalid requirement.
3. Identify the responsible agent.
4. Determine whether the parent issue has the required evidence.
5. Determine whether downstream execution has already happened.
6. Block the workflow if continuing would be unsafe.
7. Write a clear reason.
8. Request the exact correction needed.
9. Do not advance downstream.
10. Resume only after the corrected artifact or output passes validation.

Porky should not restart the full workflow if only one phase failed.

Recovery should resume from the failed phase.

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

For delegation failures:

```text
Blocked phase: Delegation
Responsible agent: Porky
Problem: Hamm subtask exists but has only a title and no complete description.
Why blocked: Hamm does not have strategy context, required artifact contract, or validation criteria.
Required next action: Porky must update the Hamm subtask issue body with a complete phase-specific description before Hamm starts.
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

Pumba should not emit noisy `BLOCKED` signals for harmless early wake-ups before Porky explicitly delegates the Notion + Delivery phase.

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
3. Confirm it is visible on the parent issue or safely reconcile it.
4. Register internal completion for the phase.
5. Request signal propagation from Babe if needed.
6. Continue safely if the artifact is valid.

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

## Failure type 3 — Artifact exists only on subtask

### Example

Hamm posts the full `content_set` artifact on the Hamm subtask, but not on the parent issue.

### Expected behavior

Porky may use the subtask artifact for reconciliation, but the workflow is not fully compliant.

Final completion requires parent issue visibility.

### Recovery action

Hamm should copy the exact same artifact and signal to the parent issue.

If Hamm cannot do it, Porky should request propagation or perform safe reconciliation according to runtime rules.

The artifact and signal must not be modified when copied.

---

## Failure type 4 — Audit artifact is incomplete

### Example

Babe posts an audit artifact without uncertainty, recommended content angles, or `critical_flags`.

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
- Critical Flags

If no critical flags exist, Babe must include:

```text
critical_flags: []
```

Porky should not synthesize strategy until the audit artifact passes validation.

---

## Failure type 5 — Babe spends run on Paperclip reference discovery

### Example

Babe wakes on an assigned research task but spends the run reading Paperclip reference documentation, inspecting unrelated issues, or exploring the environment instead of executing `audit.digital_presence`.

The process times out, crashes, or produces no audit artifact.

### Expected behavior

The workflow should not advance.

Porky should treat the research phase as incomplete.

### Recovery action

Babe must follow direct assignment execution:

1. Read the assigned issue.
2. Extract `parent_issue_id` and `subtask_issue_id`.
3. Validate required inputs.
4. Run `audit.digital_presence`.
5. Produce the audit artifact.
6. Post artifact and `AUDIT_READY` on the parent issue.
7. Stop.

Babe should not read Paperclip reference docs unless a required API or tool call fails and the reference is needed to recover.

If `audit.digital_presence` cannot complete, Babe must emit `BLOCKED` with the exact reason.

---

## Failure type 6 — Strategy artifact is missing or too vague

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

## Failure type 7 — Critical risk not resolved in strategy

### Example

Babe flags a high-severity chain positioning uncertainty.

Porky creates a strategy that uses the risky claim anyway, without excluding or safely framing it.

### Expected behavior

Porky must block before delegating to Hamm.

### Recovery action

Porky must choose one of two paths:

### Path 1 — Block for confirmation

Use this when the uncertainty prevents safe content strategy.

```text
Status: BLOCKED — Strategy requires confirmation

Reason:
Babe identified an unresolved publishing risk.

Required confirmation:
<exact confirmation needed>
```

### Path 2 — Safe strategy

Use this when content can proceed without the risky claim.

The strategy must include:

- `excluded_claims`
- `safe_framing`
- `content_rules` that forbid risky claims

Hamm may only start after the revised strategy passes validation.

---

## Failure type 8 — Content set is incomplete

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

## Failure type 9 — Content item missing required fields

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

## Failure type 10 — Thread structure is invalid

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

## Failure type 11 — Unsafe excluded claim used in content

### Example

Babe flags a risky claim.

Porky strategy includes it under `excluded_claims`.

Hamm still uses the excluded claim in a title, hook, post, thread, CTA, tag, or implied framing.

### Expected behavior

Porky must reject `CONTENT_SET_READY`.

Pumba must not persist the content.

### Recovery action

Hamm must revise the content to remove the excluded claim.

If `safe_framing` is provided, Hamm must use it.

Porky validates the revised `content_set` before delegating Pumba.

---

## Failure type 12 — Content persisted into Notes

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

After correction and verification, Pumba may emit `NOTION_SYNC_COMPLETE`.

---

## Failure type 13 — Required Notion properties are missing

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

## Failure type 14 — Research Babe page is missing

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
- Critical Flags, if present
- Source / Workflow Metadata

---

## Failure type 15 — Duplicate Notion entries

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

## Failure type 16 — Duplicate Research Babe pages

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

## Failure type 17 — Telegram summary missing

### Example

Notion persistence is complete, but no Telegram summary was sent.

### Expected behavior

Porky must not complete the workflow.

### Recovery action

Pumba must send the Telegram summary and publish the delivery artifact:

```text
ARTIFACT:
type: telegram_delivery
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
telegram_summary_sent: true
duplicate_delivery: false
timestamp:
...
```

Then emit:

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

## Failure type 18 — Duplicate Telegram summary

### Example

Pumba sends the same Telegram summary twice.

### Expected behavior

Porky should identify duplicate delivery and block final completion if duplicate delivery violates the workflow rules.

### Recovery action

Pumba should not resend valid summaries.

If a duplicate was already sent, Pumba should record that duplicate delivery occurred and avoid further sends.

---

## Failure type 19 — Downstream agent executed too early

### Example

Hamm runs before Porky has emitted a valid `SYNTHESIS_READY`.

Or Pumba runs before Hamm has produced a valid `content_set`.

### Expected behavior

The early downstream execution is invalid.

Hamm must not generate content before valid strategy exists.

Pumba must not persist content before valid `content_set` and audit artifacts exist.

### Recovery action

Porky must return to the correct phase and wait for:

For Hamm:

- valid audit artifact
- valid strategy artifact
- `SYNTHESIS_READY`
- explicit Porky delegation

For Pumba:

- valid audit artifact
- valid content_set artifact
- `CONTENT_SET_READY`
- explicit Porky delegation

Premature downstream output should not be trusted unless Porky can safely reconcile it against the correct validated inputs.

---

## Failure type 20 — Pumba wakes early before explicit delegation

### Example

Pumba wakes before Porky has delegated the Notion + Delivery phase.

### Expected behavior

Pumba must not persist anything.

Pumba must not send Telegram.

Pumba must not emit:

- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

Pumba should not emit a noisy `BLOCKED` signal unless Porky had already explicitly delegated Pumba and required inputs are missing.

### Recovery action

Pumba should exit cleanly or leave a waiting note:

```text
Status: WAITING

Reason:
Pumba is awaiting explicit delegation from Porky for the Notion + Delivery phase.
Required upstream inputs are not yet validated or not yet delegated.
```

Porky should ensure Pumba is not created until the audit and content_set artifacts are valid.

---

## Failure type 21 — Title-only subtask

### Example

Porky creates a Babe, Hamm, or Pumba subtask with only a title and no meaningful issue body.

### Expected behavior

The delegation is invalid.

The assigned specialist does not have enough context to execute deterministically.

### Recovery action

Porky must update the subtask description with:

- parent issue ID
- phase
- assigned agent
- objective
- source parent brief
- required inputs
- required output artifact
- required signal
- validation criteria
- blocking conditions
- constraints
- parent issue source-of-truth rule

If Porky cannot produce a complete subtask description, Porky must block.

---

## Failure type 22 — Wrong producer emits a signal

### Example

Pumba emits `CONTENT_SET_READY`.

### Expected behavior

Porky must treat the signal as invalid.

`CONTENT_SET_READY` can only be emitted by Hamm.

### Recovery action

The correct agent must emit the correct signal after producing the correct artifact.

---

## Failure type 23 — Malformed signal

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

## Failure type 24 — Parent issue propagation missing

### Example

Hamm emits `CONTENT_SET_READY` in the subtask but does not propagate the same signal to the parent issue.

### Expected behavior

Porky may still find the subtask signal if it scans subtasks.

However, the workflow is not fully compliant.

Final workflow completion requires parent issue visibility.

### Recovery action

Hamm must copy the exact same signal block to the parent issue.

The signal must not be regenerated or modified.

If the artifact is also missing from the parent issue, Hamm must copy the artifact too.

---

## Failure type 25 — Timeout or stalled workflow

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
4. Check whether the next specialist was delegated with a complete description.
5. If valid artifact exists, continue.
6. If no valid artifact exists, block and request the missing output.
7. If the specialist subtask is title-only or incomplete, update the description before retrying.

---

## Failure type 26 — Partial Notion persistence

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
- Is the artifact visible on the parent issue?
- Is the signal visible on the parent issue?
- Has downstream execution already occurred?
- Was downstream execution premature?
- Would retrying create duplicates?
- Can the system resume from the current state?
- Is Notion already partially updated?
- Is Telegram already sent?
- Does the current specialist subtask have a complete description?
- Is the missing work limited to one phase?

---

## Safe retry rules

Retries must be idempotent.

Before retrying, check:

- Does a valid artifact already exist?
- Does a valid Notion page already exist?
- Does a valid Research Babe page already exist?
- Was Telegram already sent?
- Is the missing work limited to one phase?
- Has the next specialist already been delegated?
- Does the delegated subtask have a complete description?

Retries should resume from the failed phase, not from the beginning.

If only a subtask description is missing, fix the subtask description before re-running the specialist.

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
- Create Hamm before strategy is valid
- Create Pumba before content_set is valid
- Treat early Pumba wake-up as workflow failure
- Let specialists guess from title-only subtasks
- Put required context only in a later comment if the issue body can be updated
- Complete the workflow with subtask-only evidence

---

## Recovery examples

### Example 1 — Missing content artifact

Problem:

Hamm emitted `CONTENT_SET_READY`, but no full content artifact exists.

Resolution:

- Porky blocks.
- Hamm publishes full `content_set`.
- Porky validates.
- Pumba runs only after validation.

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

### Example 4 — Title-only Hamm subtask

Problem:

Porky creates a Hamm subtask with only a title.

Resolution:

- Porky treats the delegation as invalid.
- Porky updates the subtask description with the strategy artifact, constraints, expected output, signal contract, and parent issue source-of-truth rule.
- Hamm executes only after the description is complete.

---

### Example 5 — Unsafe claim used in content

Problem:

Babe flags a risky claim. Porky excludes it. Hamm uses it anyway.

Resolution:

- Porky rejects `CONTENT_SET_READY`.
- Hamm removes the excluded claim.
- Hamm uses safe framing.
- Porky validates revised content before delegating Pumba.

---

## Summary

Failure recovery in Open Agentic CMO is based on a simple rule:

If the system cannot prove that the workflow state is correct, it must block.

Recovery should be:

- explicit
- phase-specific
- sequential
- idempotent
- artifact-driven
- parent-issue-visible
- safe for downstream execution

The system should never hide failure behind a successful-looking signal.

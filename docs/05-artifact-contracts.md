# Artifact Contracts

Artifacts are the structured work products produced and consumed by Open Agentic CMO agents.

If signals are the workflow transition layer, artifacts are the source of truth.

A signal declares that something is ready.  
An artifact proves that the work actually exists.

---

## Why artifacts exist

Many agent workflows fail because agents communicate through loose summaries.

Examples of weak outputs:

- “Research completed”
- “Content is ready”
- “Notion updated”
- “Strategy generated”
- “See comments above”

These statements are not enough for deterministic orchestration.

Open Agentic CMO requires every major phase to produce a structured artifact that can be validated before the workflow advances.

---

## Core principle

Every completion signal must map to a valid artifact.

| Signal | Required artifact |
|---|---|
| `AUDIT_READY` | `audit` |
| `SYNTHESIS_READY` | `strategy` |
| `CONTENT_SET_READY` | `content_set` |
| `NOTION_SYNC_COMPLETE` | `notion_content_pipeline` |
| `DELIVERY_COMPLETE` | `telegram_delivery` |

If the signal exists but the artifact is missing, the workflow must block.

If the artifact exists but the signal is missing, Porky may reconcile the workflow after validating the artifact.

---

## Artifact lifecycle

Each artifact follows this lifecycle:

1. Agent receives valid inputs.
2. Agent performs its assigned work.
3. Agent publishes the full artifact.
4. Agent validates that the artifact is complete.
5. Agent emits the completion signal.
6. Agent propagates the signal to the parent issue.
7. Porky validates the artifact before advancing.

The artifact must always exist before the completion signal.

---

## Canonical artifact format

Artifacts should use a clear block format:

```text
ARTIFACT:
type: <ARTIFACT_TYPE>
issue: <PARENT_ISSUE_ID>
subtask_issue: <SUBTASK_ISSUE_ID>
...
```

Some artifacts may not require `subtask_issue`, especially when produced directly by Porky.

The `type` field is mandatory.

The `issue` field is mandatory.

---

## Core artifact types

Open Agentic CMO currently uses the following artifact types:

| Artifact | Producer | Consumer | Purpose |
|---|---|---|---|
| `audit` | Babe | Porky, Pumba | Structured research |
| `strategy` | Porky | Hamm | Content strategy |
| `content_set` | Hamm | Porky, Pumba | Publication-ready content |
| `notion_content_pipeline` | Pumba | Porky | Notion persistence verification |
| `telegram_delivery` | Pumba | Porky | Delivery verification |

---

## Audit artifact

The `audit` artifact is produced by Babe.

It contains structured research and analysis.

### Producer

Babe

### Consumers

- Porky
- Pumba

### Related signal

`AUDIT_READY`

### Required format

```text
ARTIFACT:
type: audit
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
entity: Piggy Wallet
timestamp:
sections:

summary:
<clear concise overview>

key_signals:
- signal:
  source:
  interpretation:

inconsistencies:
- signal:
  description:
  why_it_matters:

uncertainty:
- gap:
  impact:

opportunities:
- opportunity:
  reasoning:

recommended_content_angles:
- angle:
  reasoning:
```

### Required sections

- `summary`
- `key_signals`
- `inconsistencies`
- `uncertainty`
- `opportunities`
- `recommended_content_angles`

### Validation rules

The audit artifact is valid only if:

- It is present before `AUDIT_READY`
- It has `type: audit`
- It references the correct parent issue
- It includes all required sections
- The summary is non-empty
- Key signals are present
- Uncertainty or data gaps are explicitly addressed
- Opportunities are actionable
- Recommended content angles are concrete
- It does not include final content drafts
- It is usable by Porky for strategy synthesis

### Invalid audit artifact examples

Invalid if:

- It only contains bullet notes
- It lacks uncertainty
- It contains final posts or threads
- It does not include recommended content angles
- It is not tied to the parent issue
- It is posted after `AUDIT_READY`

---

## Strategy artifact

The `strategy` artifact is produced by Porky.

It translates Babe’s audit into an actionable content strategy.

### Producer

Porky

### Consumer

Hamm

### Related signal

`SYNTHESIS_READY`

### Required format

```text
ARTIFACT:
type: strategy
issue: <PARENT_ISSUE_ID>
date_range: <DATE_RANGE>
expected_items: <NUMBER>
messaging_pillars:
- pillar:
  reasoning:
content_angles:
- angle:
  source_from_audit:
  goal:
weekly_structure:
- date:
  recommended_format:
  angle:
brand_constraints:
- constraint:
content_rules:
- rule:
```

### Required fields

- `date_range`
- `expected_items`
- `messaging_pillars`
- `content_angles`
- `weekly_structure`
- `brand_constraints`
- `content_rules`

### Validation rules

The strategy artifact is valid only if:

- It is present before `SYNTHESIS_READY`
- It has `type: strategy`
- It references the correct parent issue
- It includes a date range
- It includes expected item count
- Messaging pillars are defined
- Content angles are defined
- Weekly structure is defined
- Brand constraints are explicit
- Content rules are explicit
- It maps clearly back to the audit
- Hamm can use it without interpretation

### Invalid strategy artifact examples

Invalid if:

- It is only a summary
- It lacks date range
- It lacks expected item count
- It does not include content angles
- It gives vague direction such as “make engaging content”
- It invents unsupported product claims
- It is posted after `SYNTHESIS_READY`

---

## Content set artifact

The `content_set` artifact is produced by Hamm.

It contains the full publication-ready content set.

### Producer

Hamm

### Consumers

- Porky
- Pumba

### Related signal

`CONTENT_SET_READY`

### Required format

```text
ARTIFACT:
type: content_set
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
item_count: <NUMBER>
content_items:
- title:
  platform:
  language:
  content_type:
  scheduled_date:
  vertical:
  version:
  week:
  tags_and_mentions:
  body:
- title:
  platform:
  language:
  content_type: thread
  scheduled_date:
  vertical:
  version:
  week:
  tags_and_mentions:
  thread_id:
  tweets:
    - tweet 1
    - tweet 2
    - tweet 3
    - tweet 4
```

### Required fields per item

Every content item must include:

- `title`
- `platform`
- `language`
- `content_type`
- `scheduled_date`
- `vertical`
- `version`
- `week`
- `tags_and_mentions`

For posts:

- `body`

For threads:

- `thread_id`
- `tweets`

### Valid content types

- `post`
- `thread`

### Validation rules

The content set artifact is valid only if:

- It is present before `CONTENT_SET_READY`
- It has `type: content_set`
- It references the correct parent issue
- It references the correct Hamm subtask
- `item_count` matches the actual number of content items
- Every content item has all required fields
- Every scheduled date is valid
- `week` uses `YYYY-WXX` format
- `version` is populated
- Posts include non-empty body text
- Threads include 4–7 tweets
- Thread tweets are ordered
- Thread tweets are not duplicated
- Titles are not duplicated
- Angles are not duplicated
- Content is publication-ready
- The artifact can be consumed by Pumba without interpretation

### Invalid content set artifact examples

Invalid if:

- It only says “8 content items created”
- It contains summaries instead of full content
- It lacks body text
- It lacks tweets for threads
- It omits `version`
- It omits `week`
- It uses invalid dates
- It has duplicate titles
- It is posted after `CONTENT_SET_READY`

---

## Notion content pipeline artifact

The `notion_content_pipeline` artifact is produced or verified by Pumba.

It represents successful Notion persistence.

### Producer

Pumba

### Consumer

Porky

### Related signal

`NOTION_SYNC_COMPLETE`

### Required meaning

`NOTION_SYNC_COMPLETE` means both:

1. Content Pipeline entries were created or updated.
2. Research Babe page was created or updated.

Anything less is incomplete.

### Expected verification fields

```text
ARTIFACT:
type: notion_content_pipeline
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
content_items_persisted: <NUMBER>
research_babe_persisted: true
duplicates_found: false
notes_used_for_content: false
timestamp:
verification:
- content pipeline entries verified
- research babe page verified
- body content verified
- required properties verified
```

### Validation rules

The Notion persistence artifact is valid only if:

- All content items exist in Notion
- No duplicate content entries exist
- Required properties are populated
- Scheduled dates are correct
- Version is populated
- Week is populated
- Content Type is populated
- Posts are written in the page body
- Threads are split into ordered body blocks
- Content is not stored in Notes
- Research Babe page exists
- Research Babe includes required sections
- No duplicate Research Babe page exists

---

## Telegram delivery artifact

The `telegram_delivery` artifact is produced or verified by Pumba.

It represents successful delivery summary dispatch.

### Producer

Pumba

### Consumer

Porky

### Related signal

`DELIVERY_COMPLETE`

### Expected verification fields

```text
ARTIFACT:
type: telegram_delivery
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
telegram_summary_sent: true
duplicate_delivery: false
timestamp:
summary_includes:
- parent issue id
- date range
- total content items
- number of posts
- number of threads
- platforms used
- languages used
- Notion Content Pipeline status
- Research Babe status
```

### Validation rules

The Telegram delivery artifact is valid only if:

- Telegram summary was sent
- Delivery was not duplicated
- Summary references the parent issue
- Summary includes content persistence status
- Summary includes Research Babe status
- Summary includes counts for posts and threads
- Summary includes platforms and languages used

---

## Artifact validation ownership

Each producing agent is responsible for publishing its artifact.

Porky is responsible for validating artifacts before phase transitions.

Pumba is responsible for verifying downstream persistence artifacts.

| Artifact | Producer validates | Porky validates |
|---|---:|---:|
| `audit` | Babe | Yes |
| `strategy` | Porky | Yes |
| `content_set` | Hamm | Yes |
| `notion_content_pipeline` | Pumba | Yes |
| `telegram_delivery` | Pumba | Yes |

---

## Artifact-before-signal rule

The artifact must be visible before the completion signal is emitted.

Correct order:

1. Publish artifact.
2. Validate artifact.
3. Emit completion signal.
4. Propagate signal to parent issue.

Incorrect order:

1. Emit completion signal.
2. Publish artifact later.

The incorrect order is invalid because Porky may scan the signal before the artifact exists.

---

## Reconciliation behavior

Artifacts allow the workflow to recover from signal issues.

### Artifact exists, signal missing

Porky may:

1. Validate the artifact.
2. Register internal completion.
3. Continue workflow.
4. Request the missing signal to be propagated.

### Signal exists, artifact missing

Porky must:

1. Treat the signal as incomplete.
2. Block the workflow.
3. Request the full artifact.
4. Prevent downstream execution.

### Artifact invalid

Porky must block and request correction.

---

## Anti-patterns

Avoid these patterns:

- Emitting a signal without a full artifact
- Posting only a summary
- Saying output is “available elsewhere”
- Using informal artifact names
- Mixing artifact types in one block
- Omitting issue IDs
- Omitting required fields
- Inventing missing fields downstream
- Storing content in Notes
- Persisting incomplete artifacts
- Creating duplicate Notion pages

---

## Summary

Artifacts are the structured outputs that make Open Agentic CMO reliable.

They allow the workflow to be:

- Validated
- Reconciled
- Audited
- Reused
- Persisted safely
- Reviewed by humans

Signals tell Porky that a phase claims to be ready.

Artifacts prove that the phase is actually ready.

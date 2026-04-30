# Example Controlled E2E Test

This example shows a small controlled end-to-end test for Open Agentic CMO.

The purpose of this test is to validate the full workflow with minimal scope before running a larger weekly or campaign-level test.

Core workflow:

**Research → Strategy → Content → Notion → Delivery**

---

## When to use this test

Use this test after changing:

- Any agent `AGENTS.md`
- Signal contracts
- Artifact contracts
- Porky validation gates
- Pumba Notion persistence rules
- Notion database schema
- Telegram delivery behavior

This test is intentionally small so failures are easier to diagnose.

---

## Test objective

Validate that the workflow can autonomously:

1. Generate a structured Babe audit artifact.
2. Generate a structured Porky strategy artifact.
3. Generate a structured Hamm content_set artifact.
4. Persist content correctly into the Notion Content Pipeline.
5. Persist Babe research into a Notion page titled `Research Babe`.
6. Send a Telegram summary.
7. Emit and propagate all required signals.
8. Stop only after `DELIVERY_COMPLETE`.

---

## Test scope

Date range:

**May 11, 2026 → May 13, 2026**

Expected output:

- 3 total content items
- Exactly 1 item per scheduled date
- 1 X thread
- 1 X post
- 1 LinkedIn post
- 1 Research Babe page
- 1 Telegram summary

Agents involved:

- Porky
- Babe
- Hamm
- Pumba

Excluded:

- Piglet
- Image generation
- Direct social publishing

---

## Parent issue template

Copy this into a new parent issue assigned to Porky.

```text
TITLE: Controlled E2E Test — Artifact-Based Pipeline + Research Babe Notion Page

DESCRIPTION:

Run a small controlled end-to-end validation of the Open Agentic CMO core content pipeline.

This test validates the artifact-based contracts and hard validation gates before running a larger weekly E2E test.

Pipeline:

Research → Strategy → Content → Notion → Delivery

Agents involved:

- Porky: orchestration, strategy synthesis, validation gates
- Babe: research/audit artifact
- Hamm: structured content_set artifact
- Pumba: Notion persistence + Research Babe page + Telegram delivery

Piglet is intentionally excluded from this test.

OBJECTIVE

Validate that the workflow can autonomously:

1. Generate a structured Babe audit artifact
2. Generate a structured Porky strategy artifact
3. Generate a structured Hamm content_set artifact
4. Persist content correctly into Notion Content Pipeline
5. Persist Babe research into a Notion page titled Research Babe
6. Send Telegram summary
7. Emit and propagate all required signals
8. Stop only after DELIVERY_COMPLETE

DATE RANGE

May 11, 2026 → May 13, 2026

Total days: 3

Expected output:

- 3 total content items
- Exactly 1 item per scheduled date
- 1 X thread
- 1 X post
- 1 LinkedIn post

CONTENT REQUIREMENTS

Each content item must include:

- title
- platform
- language
- content_type
- scheduled_date
- vertical
- version
- week
- tags_and_mentions

For posts:

- body

For threads:

- thread_id
- tweets array
- 4–7 tweets
- logical progression
- no duplicate tweets

RESEARCH REQUIREMENTS

Babe must publish a full audit artifact before AUDIT_READY.

Required format:

ARTIFACT:
type: audit
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
entity: Piggy Wallet
timestamp:
sections:

Required sections:

- summary
- key_signals
- inconsistencies
- uncertainty
- opportunities
- recommended_content_angles

STRATEGY REQUIREMENTS

Porky must publish a full strategy artifact before SYNTHESIS_READY.

Required format:

ARTIFACT:
type: strategy
issue: <PARENT_ISSUE_ID>
date_range:
expected_items:
messaging_pillars:
content_angles:
weekly_structure:
brand_constraints:
content_rules:

CONTENT ARTIFACT REQUIREMENTS

Hamm must publish a full content_set artifact before CONTENT_SET_READY.

Required format:

ARTIFACT:
type: content_set
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
item_count: 3
content_items:

Do not emit CONTENT_SET_READY unless the full artifact is visible in the Hamm subtask.

NOTION REQUIREMENTS

Pumba must persist BOTH:

1. Content Pipeline entries
2. Research Babe page

Content Pipeline:

For each content item, set:

- Title
- Platform
- Status = Scheduled
- Scheduled Date
- Vertical
- Version
- Week
- Content Type

Body rules:

- Post content must be placed in page body
- Thread tweets must be placed as ordered page body blocks
- Content must NOT be placed in Notes

Research Babe:

Create or update a page titled:

Research Babe — <PARENT_ISSUE_ID> — May 11–13, 2026

The page body must include:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Source / Workflow Metadata

VALIDATION GATES

Porky must validate before advancing:

1. AUDIT_READY is valid only if audit artifact exists and passes validation
2. SYNTHESIS_READY is valid only if strategy artifact exists and passes validation
3. CONTENT_SET_READY is valid only if content_set artifact exists and passes validation
4. NOTION_SYNC_COMPLETE is valid only if:
   - all 3 content items exist in Notion
   - no duplicates exist
   - Version is populated
   - Week is populated
   - Content Type is populated
   - content is in body, not Notes
   - Research Babe page exists
5. DELIVERY_COMPLETE is valid only if Telegram summary was sent once

EXPECTED SIGNALS

SIGNAL:
type: AUDIT_READY
producer: Babe
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: audit
timestamp:

SIGNAL:
type: SYNTHESIS_READY
producer: Porky
issue: <PARENT_ISSUE_ID>
status: COMPLETE
artifact: strategy
timestamp:

SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: content_set
timestamp:

SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: notion_content_pipeline
timestamp:

SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: telegram_delivery
timestamp:

FAILURE CONDITIONS

The test fails if:

- any required artifact is missing
- any signal is emitted before its artifact
- Hamm emits only a summary instead of full content_items
- Pumba persists content into Notes
- Version is empty
- Week is empty
- Research Babe page is missing
- duplicate Notion entries are created
- Telegram summary is sent more than once
- Porky advances despite validation errors

FINAL OUTPUT REQUIRED FROM PORKY

At completion, post a final summary including:

- parent issue id
- date range
- total content items
- number of posts
- number of threads
- platforms used
- audit artifact validation result
- strategy artifact validation result
- content_set artifact validation result
- Content Pipeline Notion validation result
- Research Babe Notion validation result
- Telegram delivery result
- emitted signals
- any issues found and resolved

Do not close the workflow unless all validation gates pass.
```

---

## Expected artifact sequence

A successful controlled test should produce artifacts in this order:

1. `ARTIFACT: type: audit`
2. `SIGNAL: AUDIT_READY`
3. `ARTIFACT: type: strategy`
4. `SIGNAL: SYNTHESIS_READY`
5. `ARTIFACT: type: content_set`
6. `SIGNAL: CONTENT_SET_READY`
7. Notion Content Pipeline entries
8. Research Babe page
9. `SIGNAL: NOTION_SYNC_COMPLETE`
10. Telegram summary
11. `SIGNAL: DELIVERY_COMPLETE`

---

## Expected final state

At the end of the test:

- There are 3 content pages in the Notion Content Pipeline.
- There is 1 Research Babe page.
- No content is stored in Notes.
- All content pages have `Version`.
- All content pages have `Week`.
- All content pages have `Content Type`.
- The X thread is split into ordered body blocks.
- Telegram summary was sent once.
- Porky posted a final summary.
- Workflow stopped only after `DELIVERY_COMPLETE`.

---

## Manual validation checklist

### Babe

- [ ] Published `ARTIFACT: type: audit`
- [ ] Included summary
- [ ] Included key signals
- [ ] Included inconsistencies
- [ ] Included uncertainty / data gaps
- [ ] Included opportunities
- [ ] Included recommended content angles
- [ ] Emitted `AUDIT_READY`
- [ ] Propagated `AUDIT_READY` to the parent issue

### Porky

- [ ] Validated Babe audit artifact
- [ ] Published `ARTIFACT: type: strategy`
- [ ] Included date range
- [ ] Included expected item count
- [ ] Included messaging pillars
- [ ] Included content angles
- [ ] Included weekly structure
- [ ] Included brand constraints
- [ ] Included content rules
- [ ] Emitted `SYNTHESIS_READY`
- [ ] Did not advance with invalid artifacts

### Hamm

- [ ] Published `ARTIFACT: type: content_set`
- [ ] Included `item_count: 3`
- [ ] Included 3 full content items
- [ ] Included 1 X thread
- [ ] Included 1 X post
- [ ] Included 1 LinkedIn post
- [ ] Every item has title
- [ ] Every item has platform
- [ ] Every item has language
- [ ] Every item has content_type
- [ ] Every item has scheduled_date
- [ ] Every item has vertical
- [ ] Every item has version
- [ ] Every item has week
- [ ] Every item has tags_and_mentions
- [ ] Posts include body
- [ ] Thread includes thread_id
- [ ] Thread includes 4–7 tweets
- [ ] Emitted `CONTENT_SET_READY`
- [ ] Propagated `CONTENT_SET_READY` to the parent issue

### Pumba

- [ ] Persisted 3 content items into Notion
- [ ] Created or updated Research Babe page
- [ ] Set Title
- [ ] Set Platform
- [ ] Set Status
- [ ] Set Scheduled Date
- [ ] Set Vertical
- [ ] Set Version
- [ ] Set Week
- [ ] Set Content Type
- [ ] Set Thread ID for thread
- [ ] Put post body in page body
- [ ] Put thread tweets as ordered body blocks
- [ ] Did not store content in Notes
- [ ] Sent Telegram summary once
- [ ] Emitted `NOTION_SYNC_COMPLETE`
- [ ] Emitted `DELIVERY_COMPLETE`
- [ ] Propagated both signals to the parent issue

---

## Passing criteria

The controlled E2E test passes only if:

- All required artifacts exist.
- Artifacts appear before completion signals.
- All required signals exist.
- Signals are propagated to the parent issue.
- Porky validates every phase.
- Pumba persists both content and Research Babe.
- Notion content is in page body, not Notes.
- Required Notion properties are populated.
- Telegram summary is sent once.
- Porky posts a final summary.
- No duplicate content entries exist.
- No duplicate Research Babe pages exist.

---

## What to do if the test fails

Use `docs/09-failure-recovery.md`.

Do not restart the entire workflow unless necessary.

Identify the failed phase and resume from the smallest safe point.

Examples:

- If Babe audit is missing, fix Babe only.
- If Hamm content_set is missing, fix Hamm only.
- If Notion persistence is wrong, fix Pumba only.
- If Telegram summary is missing, resume delivery only.

The goal is safe recovery, not full regeneration.

# E2E Testing

End-to-end testing is how Open Agentic CMO validates that the full multi-agent workflow works correctly from start to finish.

The goal of E2E testing is not only to check that agents produce outputs.

The goal is to verify that the entire system behaves deterministically, respects contracts, persists clean data, and stops safely when something is invalid.

---

## Core workflow under test

The full workflow is:

**Research → Strategy → Content → Notion → Delivery**

A successful E2E test validates that:

1. Babe produces a valid `audit` artifact.
2. Porky produces a valid `strategy` artifact.
3. Hamm produces a valid `content_set` artifact.
4. Pumba persists content into the Notion Content Pipeline.
5. Pumba persists Babe research into `Research Babe`.
6. Pumba sends a Telegram summary.
7. Porky validates every phase.
8. The workflow stops only after `DELIVERY_COMPLETE`.

---

## Why E2E testing matters

Multi-agent systems can appear to work while silently failing.

Common hidden failures include:

- Completion signals emitted before artifacts exist
- Agents posting summaries instead of structured artifacts
- Downstream agents running too early
- Content being persisted into `Notes`
- Missing Notion properties such as `Version` or `Week`
- Threads being merged into one unstructured block
- Research context being lost
- Duplicate Notion pages
- Duplicate Telegram summaries
- Porky advancing despite validation failures

E2E testing catches these failures.

---

## Recommended testing strategy

Use progressive testing.

Do not start with a large campaign test.

Recommended order:

1. Controlled 3-day test
2. Weekly 7-day test
3. Extended 11-day test
4. Failure-case tests
5. Regression tests after agent changes

---

## Test 1 — Controlled 3-day test

Use this test after changing:

- Agent instructions
- Signal contracts
- Artifact contracts
- Notion persistence rules
- Porky validation gates

### Goal

Validate the full artifact-based pipeline with minimal scope.

### Date range

May 11, 2026 → May 13, 2026

### Expected output

- 3 total content items
- Exactly 1 item per scheduled date
- 1 X thread
- 1 X post
- 1 LinkedIn post
- 1 Research Babe page
- 1 Telegram summary

### Required agents

- Porky
- Babe
- Hamm
- Pumba

Piglet is excluded.

---

## Controlled 3-day test issue

Copy this into a new parent issue assigned to Porky:

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

## Test 2 — Weekly 7-day test

Use this after the controlled 3-day test passes.

### Goal

Validate a normal weekly content workflow.

### Expected output

- 7 total content items
- Exactly 1 item per scheduled date
- At least 3 X threads
- Mix of X and LinkedIn
- Research Babe page
- Telegram summary

### What it validates

- Weekly distribution
- Multiple threads
- Multi-platform mix
- Week field generation
- Notion deduplication
- Research traceability

---

## Test 3 — Extended 11-day test

Use this for larger campaign validation.

### Goal

Validate a longer content range and stronger scheduling behavior.

### Expected output

- 11 total content items
- Exactly 1 item per scheduled date
- At least 3 X threads per week
- No missing dates
- No duplicates
- Research Babe page
- Telegram summary

### What it validates

- Longer range execution
- Date coverage
- Thread ratio
- Notion scaling
- Idempotency across more items

---

## Validation checklist

After every E2E test, verify the following manually or automatically.

### Babe

- `ARTIFACT: type: audit` exists
- Artifact appears before `AUDIT_READY`
- Required sections are present
- Uncertainty is included
- Recommended content angles are included
- Signal is propagated to parent issue

### Porky

- `ARTIFACT: type: strategy` exists
- Artifact appears before `SYNTHESIS_READY`
- Date range is present
- Expected item count is present
- Messaging pillars are present
- Content angles are present
- Strategy maps to audit
- Porky does not advance with invalid artifacts

### Hamm

- `ARTIFACT: type: content_set` exists
- Artifact appears before `CONTENT_SET_READY`
- `item_count` matches actual items
- Every content item has required fields
- Posts include `body`
- Threads include `thread_id` and `tweets`
- Thread tweets are ordered
- No duplicate titles
- Signal is propagated to parent issue

### Pumba

- Content Pipeline pages exist
- Research Babe page exists
- No duplicate content pages
- No duplicate Research Babe page
- `Version` is populated
- `Week` is populated
- `Content Type` is populated
- Content is in page body
- Content is not in `Notes`
- Threads are split into ordered body blocks
- Telegram summary is sent once
- Both Pumba signals are propagated

---

## Failure-case tests

Failure tests are as important as successful E2E tests.

They validate that the system blocks safely.

---

### Failure test 1 — Signal without artifact

Simulate:

- Hamm emits `CONTENT_SET_READY`
- No full `content_set` artifact exists

Expected behavior:

- Porky treats signal as incomplete
- Porky blocks
- Pumba does not run
- Porky requests full artifact from Hamm

---

### Failure test 2 — Content in Notes

Simulate:

- Pumba persists content into `Notes` instead of page body

Expected behavior:

- Porky rejects `NOTION_SYNC_COMPLETE`
- Workflow blocks
- Pumba must fix Notion body structure

---

### Failure test 3 — Missing Research Babe

Simulate:

- Pumba persists content pages
- Pumba skips Research Babe

Expected behavior:

- Porky rejects `NOTION_SYNC_COMPLETE`
- Workflow blocks
- Pumba must create or update Research Babe page

---

### Failure test 4 — Missing Version or Week

Simulate:

- Content pages exist
- `Version` or `Week` is empty

Expected behavior:

- Porky rejects Notion validation
- Workflow blocks
- Pumba must update missing properties

---

### Failure test 5 — Duplicate Notion entries

Simulate:

- Same content item appears twice in Notion

Expected behavior:

- Porky rejects Notion validation
- Workflow blocks
- Pumba must deduplicate safely

---

## Regression testing

Run a controlled E2E test after changing:

- Any `AGENTS.md`
- Signal contract
- Artifact contract
- Notion schema
- Pumba persistence logic
- Porky validation gates
- Delivery behavior

Recommended regression test:

- 3-day controlled E2E
- 1 X thread
- 1 X post
- 1 LinkedIn post
- Research Babe required

---

## Passing criteria

An E2E test passes only when:

- All required artifacts exist
- All required signals exist
- Artifacts appear before completion signals
- Porky validates each phase
- Notion content is correctly persisted
- Research Babe is created or updated
- Telegram summary is sent once
- No duplicates exist
- No content is stored in Notes
- Final Porky summary confirms all gates passed

---

## Summary

E2E testing confirms that Open Agentic CMO works as a system, not just as a collection of agents.

A valid test proves that the workflow is:

- deterministic
- artifact-driven
- signal-driven
- validation-first
- idempotent
- recoverable
- human-reviewable

The most important rule is simple:

If the system cannot prove correctness, it must block.

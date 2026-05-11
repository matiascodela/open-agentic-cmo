# Example Controlled E2E Test

This example shows a small controlled end-to-end test for Open Agentic CMO.

The purpose of this test is to validate the full workflow with minimal scope before running a larger weekly or campaign-level test.

Unlike earlier scripted tests, this controlled test uses a realistic, high-level parent issue.

The parent issue should not need to include every internal artifact, signal, validation gate, or subtask template.

Porky is responsible for expanding the vague operator prompt into complete phase-specific subtasks.

Core workflow:

```text
Research → Strategy → Content → Notion → Delivery
```

Canonical agent execution order:

```text
Porky → Babe → Porky → Hamm → Porky → Pumba → Porky
```

---

## When to use this test

Use this test after changing:

- Any agent `AGENTS.md`
- Any agent `HEARTBEAT.md`
- Signal contracts
- Artifact contracts
- Porky validation gates
- Porky subtask creation rules
- Babe research execution rules
- Hamm content generation rules
- Pumba Notion persistence rules
- Notion database schema
- Telegram delivery behavior

This test is intentionally small so failures are easier to diagnose.

---

## Test objective

Validate that the workflow can autonomously:

1. Receive a high-level operator prompt.
2. Have Porky create only the Babe research subtask first.
3. Have Porky create complete subtask descriptions instead of title-only tasks.
4. Generate a structured Babe audit artifact.
5. Generate a structured Porky strategy artifact.
6. Generate a structured Hamm `content_set` artifact.
7. Persist content correctly into the Notion Content Pipeline.
8. Persist Babe research into a Notion page titled `Research Babe`.
9. Send a Telegram summary.
10. Emit all required signals.
11. Post all final artifacts and signals on the parent issue.
12. Stop only after `DELIVERY_COMPLETE`.

---

## Test scope

Date range:

```text
May 11, 2026 → May 13, 2026
```

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
TITLE: Controlled E2E Test — Piggy Wallet Content Workflow

DESCRIPTION:

Run a controlled Piggy Wallet content workflow for May 11–13, 2026.

Create 3 content items:
- 1 X thread
- 1 X post
- 1 LinkedIn post

Save the content and the research context to Notion, then send the Telegram summary.
```

This prompt is intentionally simple.

The test passes only if Porky expands this prompt into complete, deterministic workflow execution.

---

## Expected workflow behavior

A successful controlled E2E test should follow this exact sequence:

1. Porky starts the parent workflow.
2. Porky creates only the Babe research subtask.
3. Babe subtask has a complete issue description.
4. Hamm is not created yet.
5. Pumba is not created yet.
6. Babe executes the assigned research task directly.
7. Babe publishes the full `audit` artifact on the parent issue.
8. Babe emits `AUDIT_READY` on the parent issue.
9. Porky validates the audit artifact.
10. Porky publishes the `strategy` artifact on the parent issue.
11. Porky emits `SYNTHESIS_READY` on the parent issue.
12. Porky creates the Hamm content subtask.
13. Hamm subtask has a complete issue description including strategy context.
14. Hamm publishes the full `content_set` artifact on the parent issue.
15. Hamm emits `CONTENT_SET_READY` on the parent issue.
16. Porky validates the content set.
17. Porky creates the Pumba Notion + Delivery subtask.
18. Pumba subtask has a complete issue description including audit and content_set context.
19. Pumba persists content into the Notion Content Pipeline.
20. Pumba persists Babe research into Research Babe.
21. Pumba publishes `notion_content_pipeline` artifact on the parent issue.
22. Pumba emits `NOTION_SYNC_COMPLETE` on the parent issue.
23. Pumba sends Telegram summary.
24. Pumba publishes `telegram_delivery` artifact on the parent issue.
25. Pumba emits `DELIVERY_COMPLETE` on the parent issue.
26. Porky validates final state.
27. Porky posts final summary.
28. Workflow stops.

---

## Expected subtask behavior

Porky must not create title-only subtasks.

Every delegated subtask must include a complete issue description.

A complete subtask description includes:

- `parent_issue_id`
- phase name
- assigned agent
- objective
- workflow context
- source parent brief
- required inputs
- required output artifact
- expected signal
- validation criteria
- blocking conditions
- explicit constraints
- parent issue source-of-truth rule
- propagation requirement

If any subtask is title-only, the test fails.

---

## Expected artifact sequence

A successful controlled test should produce artifacts and signals in this order:

```text
1. ARTIFACT: type: audit
2. SIGNAL: AUDIT_READY
3. ARTIFACT: type: strategy
4. SIGNAL: SYNTHESIS_READY
5. ARTIFACT: type: content_set
6. SIGNAL: CONTENT_SET_READY
7. ARTIFACT: type: notion_content_pipeline
8. SIGNAL: NOTION_SYNC_COMPLETE
9. ARTIFACT: type: telegram_delivery
10. SIGNAL: DELIVERY_COMPLETE
```

All final artifacts and signals must be visible on the parent issue.

Subtask-only artifacts or signals are not enough for final completion.

---

## Required audit artifact

Babe must publish a full audit artifact before `AUDIT_READY`.

Required format:

```text
ARTIFACT:
type: audit
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
entity: Piggy Wallet
timestamp:
sections:
  summary:
  key_signals:
  inconsistencies:
  uncertainty:
  opportunities:
  recommended_content_angles:
  critical_flags:
```

Required sections:

- `summary`
- `key_signals`
- `inconsistencies`
- `uncertainty`
- `opportunities`
- `recommended_content_angles`
- `critical_flags`

If no critical flags exist, Babe must include:

```text
critical_flags: []
```

Babe must not emit `AUDIT_READY` before the full artifact exists on the parent issue.

---

## Required strategy artifact

Porky must publish a full strategy artifact before `SYNTHESIS_READY`.

Required format:

```text
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
```

If Babe’s audit includes critical publishing risks, Porky’s strategy must also include:

```text
excluded_claims:
safe_framing:
```

Porky must not emit `SYNTHESIS_READY` before the strategy artifact exists on the parent issue.

Porky must not delegate Hamm before the strategy artifact is valid.

---

## Required content_set artifact

Hamm must publish a full content set artifact before `CONTENT_SET_READY`.

Required format:

```text
ARTIFACT:
type: content_set
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
item_count: 3
content_items:
```

Each content item must include:

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

Thread requirements:

- 4–7 tweets
- ordered tweet array
- logical progression
- no duplicate tweets

Hamm must respect:

- `brand_constraints`
- `content_rules`
- `excluded_claims`, if present
- `safe_framing`, if present

Hamm must not emit `CONTENT_SET_READY` before the full artifact exists on the parent issue.

---

## Required Notion persistence

Pumba must persist both:

1. Content Pipeline entries
2. Research Babe page

### Content Pipeline

For each content item, set:

- `Title`
- `Platform`
- `Status = Scheduled`
- `Scheduled Date`
- `Vertical`
- `Version`
- `Week`
- `Content Type`

For threads, also set:

- `Thread ID`, if the database supports it
- `Thread Order`, if applicable and supported

Body rules:

- Post content must be placed in the page body.
- Thread tweets must be placed as ordered page body blocks.
- Content must not be placed in `Notes`.

### Research Babe

Create or update a page titled:

```text
Research Babe — <PARENT_ISSUE_ID> — May 11–13, 2026
```

The page body must include:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Critical Flags, if present
- Source / Workflow Metadata

---

## Required Pumba artifacts

Before `NOTION_SYNC_COMPLETE`, Pumba must publish:

```text
ARTIFACT:
type: notion_content_pipeline
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
content_items_persisted:
expected_content_items:
research_babe_persisted:
duplicates_found:
notes_used_for_content:
timestamp:
verification:
```

Before `DELIVERY_COMPLETE`, Pumba must publish:

```text
ARTIFACT:
type: telegram_delivery
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
telegram_summary_sent:
duplicate_delivery:
timestamp:
summary_includes:
verification:
```

Both artifacts must be posted on the parent issue.

---

## Expected signals

### AUDIT_READY

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: audit
timestamp:
```

### SYNTHESIS_READY

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
issue: <PARENT_ISSUE_ID>
status: COMPLETE
artifact: strategy
timestamp:
```

### CONTENT_SET_READY

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: content_set
timestamp:
```

### NOTION_SYNC_COMPLETE

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: notion_content_pipeline
timestamp:
```

### DELIVERY_COMPLETE

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
issue: <PARENT_ISSUE_ID>
subtask_issue: <PUMBA_SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: telegram_delivery
timestamp:
```

---

## Manual validation checklist

### Parent issue

- [ ] Parent issue contains `ARTIFACT: type: audit`
- [ ] Parent issue contains `SIGNAL: AUDIT_READY`
- [ ] Parent issue contains `ARTIFACT: type: strategy`
- [ ] Parent issue contains `SIGNAL: SYNTHESIS_READY`
- [ ] Parent issue contains `ARTIFACT: type: content_set`
- [ ] Parent issue contains `SIGNAL: CONTENT_SET_READY`
- [ ] Parent issue contains `ARTIFACT: type: notion_content_pipeline`
- [ ] Parent issue contains `SIGNAL: NOTION_SYNC_COMPLETE`
- [ ] Parent issue contains `ARTIFACT: type: telegram_delivery`
- [ ] Parent issue contains `SIGNAL: DELIVERY_COMPLETE`
- [ ] Final Porky summary exists

### Subtasks

- [ ] Babe subtask exists first
- [ ] Babe subtask has a complete description
- [ ] Hamm subtask is not created before Babe audit and Porky strategy are valid
- [ ] Hamm subtask has a complete description
- [ ] Pumba subtask is not created before Hamm content_set is valid
- [ ] Pumba subtask has a complete description
- [ ] No title-only subtasks exist

### Babe

- [ ] Executes the assigned research task directly
- [ ] Does not spend the run on broad Paperclip reference discovery
- [ ] Publishes `ARTIFACT: type: audit`
- [ ] Includes summary
- [ ] Includes key signals
- [ ] Includes inconsistencies
- [ ] Includes uncertainty / data gaps
- [ ] Includes opportunities
- [ ] Includes recommended content angles
- [ ] Includes `critical_flags`, even if empty
- [ ] Emits `AUDIT_READY`
- [ ] Posts artifact and signal on the parent issue

### Porky

- [ ] Validates Babe audit artifact
- [ ] Publishes `ARTIFACT: type: strategy`
- [ ] Includes date range
- [ ] Includes expected item count
- [ ] Includes messaging pillars
- [ ] Includes content angles
- [ ] Includes weekly structure
- [ ] Includes brand constraints
- [ ] Includes content rules
- [ ] Includes `excluded_claims` if audit contains publishing risks
- [ ] Includes `safe_framing` if audit contains publishing risks
- [ ] Emits `SYNTHESIS_READY`
- [ ] Does not advance with invalid artifacts
- [ ] Does not create downstream subtasks early

### Hamm

- [ ] Starts only after Porky explicitly delegates content
- [ ] Consumes Porky’s strategy artifact
- [ ] Publishes `ARTIFACT: type: content_set`
- [ ] Includes `item_count: 3`
- [ ] Includes 3 full content items
- [ ] Includes 1 X thread
- [ ] Includes 1 X post
- [ ] Includes 1 LinkedIn post
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
- [ ] Content avoids `excluded_claims`
- [ ] Content uses `safe_framing` when required
- [ ] Emits `CONTENT_SET_READY`
- [ ] Posts artifact and signal on the parent issue

### Pumba

- [ ] Starts only after Porky explicitly delegates Notion + Delivery
- [ ] Does not run early
- [ ] Does not emit noisy `BLOCKED` before explicit delegation
- [ ] Persists 3 content items into Notion
- [ ] Creates or updates Research Babe page
- [ ] Sets Title
- [ ] Sets Platform
- [ ] Sets Status
- [ ] Sets Scheduled Date
- [ ] Sets Vertical
- [ ] Sets Version
- [ ] Sets Week
- [ ] Sets Content Type
- [ ] Sets Thread ID for thread if supported
- [ ] Puts post body in page body
- [ ] Puts thread tweets as ordered body blocks
- [ ] Does not store content in Notes
- [ ] Publishes `ARTIFACT: type: notion_content_pipeline`
- [ ] Emits `NOTION_SYNC_COMPLETE`
- [ ] Sends Telegram summary once
- [ ] Publishes `ARTIFACT: type: telegram_delivery`
- [ ] Emits `DELIVERY_COMPLETE`
- [ ] Posts all final artifacts and signals on the parent issue

---

## Passing criteria

The controlled E2E test passes only if:

- Porky delegates agents sequentially.
- Porky creates only Babe first.
- Hamm is created only after audit and strategy validation.
- Pumba is created only after content_set validation.
- No title-only subtasks exist.
- All required artifacts exist.
- Artifacts appear before completion signals.
- All required signals exist.
- All final artifacts and signals appear on the parent issue.
- Porky validates every phase.
- Pumba persists both content and Research Babe.
- Notion content is in page body, not Notes.
- Required Notion properties are populated.
- Telegram summary is sent once.
- Porky posts a final summary.
- No duplicate content entries exist.
- No duplicate Research Babe pages exist.

---

## Failure conditions

The test fails if:

- Porky creates Hamm before Babe completes and strategy is valid.
- Porky creates Pumba before Hamm completes and content_set is valid.
- Any subtask is title-only.
- Any required artifact is missing.
- Any completion signal is emitted before its artifact.
- Any final artifact exists only on a subtask.
- Babe omits `critical_flags`.
- Babe spends the run on broad Paperclip reference discovery instead of producing the audit.
- Porky emits `SYNTHESIS_READY` without a strategy artifact.
- Porky fails to include `excluded_claims` or `safe_framing` when audit risks require them.
- Hamm emits only a summary instead of full content_items.
- Hamm uses an excluded claim.
- Pumba persists content into Notes.
- Pumba skips Research Babe.
- Pumba emits `NOTION_SYNC_COMPLETE` without Notion verification.
- Pumba emits `DELIVERY_COMPLETE` without Telegram verification.
- Version is empty.
- Week is empty.
- Research Babe page is missing.
- Duplicate Notion entries are created.
- Telegram summary is sent more than once.
- Porky advances despite validation errors.

---

## What to do if the test fails

Use:

```text
docs/09-failure-recovery.md
```

Do not restart the entire workflow unless necessary.

Identify the failed phase and resume from the smallest safe point.

Examples:

- If Babe audit is missing, fix Babe only.
- If Babe omitted `critical_flags`, fix Babe only.
- If Porky strategy is missing, fix Porky only.
- If Hamm content_set is missing, fix Hamm only.
- If Hamm used an excluded claim, fix Hamm only.
- If Notion persistence is wrong, fix Pumba only.
- If Research Babe is missing, fix Pumba only.
- If Telegram summary is missing, resume delivery only.
- If a subtask is title-only, update that subtask description before rerunning the specialist.

The goal is safe recovery, not full regeneration.

---

## Summary

This controlled E2E test verifies that Open Agentic CMO works as a system, not just as a collection of agents.

A passing test proves that the workflow is:

- deterministic
- sequential
- artifact-driven
- signal-driven
- validation-first
- context-rich
- idempotent
- recoverable
- human-reviewable

The most important rule is simple:

If the system cannot prove correctness, it must block.

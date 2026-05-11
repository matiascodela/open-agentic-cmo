# E2E Testing

End-to-end testing is how Open Agentic CMO validates that the full multi-agent workflow works correctly from start to finish.

The goal of E2E testing is not only to check that agents produce outputs.

The goal is to verify that the entire system behaves deterministically, respects contracts, expands vague operator prompts into complete subtasks, persists clean data, and stops safely when something is invalid.

---

## Core workflow under test

The full workflow is:

```text
Research → Strategy → Content → Notion → Delivery
```

The canonical agent execution order is:

```text
Porky → Babe → Porky → Hamm → Porky → Pumba → Porky
```

A successful E2E test validates that:

1. Porky creates only the Babe subtask first.
2. Porky creates a complete Babe subtask description, not a title-only task.
3. Babe produces a valid `audit` artifact.
4. Babe posts the `audit` artifact and `AUDIT_READY` signal on the parent issue.
5. Porky validates Babe’s audit.
6. Porky produces a valid `strategy` artifact.
7. Porky emits `SYNTHESIS_READY` on the parent issue.
8. Porky creates Hamm only after audit and strategy validation pass.
9. Porky creates a complete Hamm subtask description.
10. Hamm produces a valid `content_set` artifact.
11. Hamm posts the `content_set` artifact and `CONTENT_SET_READY` signal on the parent issue.
12. Porky validates Hamm’s content.
13. Porky creates Pumba only after content validation passes.
14. Porky creates a complete Pumba subtask description.
15. Pumba persists content into the Notion Content Pipeline.
16. Pumba persists Babe research into `Research Babe`.
17. Pumba sends a Telegram summary.
18. Pumba posts final persistence and delivery artifacts on the parent issue.
19. Porky validates every phase.
20. The workflow stops only after `DELIVERY_COMPLETE`.

---

## Why E2E testing matters

Multi-agent systems can appear to work while silently failing.

Common hidden failures include:

- Completion signals emitted before artifacts exist
- Agents posting summaries instead of structured artifacts
- Downstream agents running too early
- Porky creating all subtasks at workflow initialization
- Porky creating title-only subtasks
- Subtasks missing source context, strategy, artifact requirements, or validation criteria
- Agents posting final artifacts only on subtasks
- Content being persisted into `Notes`
- Missing Notion properties such as `Version` or `Week`
- Threads being merged into one unstructured block
- Research context being lost
- `Research Babe` page missing
- Duplicate Notion pages
- Duplicate Telegram summaries
- Porky advancing despite validation failures
- Babe spending the run on Paperclip reference discovery instead of completing the audit
- Pumba waking early and producing noisy `BLOCKED` signals before it was explicitly delegated

E2E testing catches these failures.

---

## Recommended testing strategy

Use progressive testing.

Do not start with a large campaign test.

Recommended order:

1. Controlled 3-day test with a vague operator prompt
2. Weekly 7-day test with a normal production-style prompt
3. Extended 11-day test
4. Failure-case tests
5. Regression tests after agent changes

The key rule: the system should pass with a realistic, high-level parent issue.

The parent issue should not need to spell out every internal artifact, signal, and phase rule. Porky must expand the vague prompt into rich phase-specific subtasks.

---

## Test 1 — Controlled 3-day test

Use this test after changing:

- Agent instructions
- Signal contracts
- Artifact contracts
- Sequential delegation rules
- Subtask creation rules
- Notion persistence rules
- Porky validation gates
- Babe research execution rules
- Pumba delivery rules

### Goal

Validate the full artifact-based pipeline with minimal scope.

This test should confirm that a simple operator prompt is enough for Porky to:

- infer the workflow
- delegate only the next valid phase
- create complete subtask descriptions
- validate artifacts
- prevent premature downstream execution
- persist final outputs correctly

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

The test passes only if Porky expands this vague prompt into complete phase-specific execution.

---

## Controlled 3-day expected behavior

The expected workflow behavior is:

1. Porky starts the parent workflow.
2. Porky creates only the Babe research subtask.
3. Babe subtask has a complete description.
4. Hamm is not created yet.
5. Pumba is not created yet.
6. Babe publishes the full `audit` artifact on the parent issue.
7. Babe emits `AUDIT_READY` on the parent issue.
8. Porky validates the audit.
9. Porky publishes the `strategy` artifact on the parent issue.
10. Porky emits `SYNTHESIS_READY` on the parent issue.
11. Porky creates the Hamm content subtask.
12. Hamm subtask has a complete description including strategy context.
13. Hamm publishes the full `content_set` artifact on the parent issue.
14. Hamm emits `CONTENT_SET_READY` on the parent issue.
15. Porky validates the content set.
16. Porky creates the Pumba Notion + Delivery subtask.
17. Pumba subtask has a complete description including audit and content_set context.
18. Pumba persists content into Notion Content Pipeline.
19. Pumba persists Babe research into Research Babe.
20. Pumba publishes `notion_content_pipeline` artifact on the parent issue.
21. Pumba emits `NOTION_SYNC_COMPLETE` on the parent issue.
22. Pumba sends Telegram summary.
23. Pumba publishes `telegram_delivery` artifact on the parent issue.
24. Pumba emits `DELIVERY_COMPLETE` on the parent issue.
25. Porky validates final state.
26. Porky posts final summary.

---

## Controlled 3-day pass/fail criteria

The controlled test passes only if:

### Porky

- Creates only Babe first
- Does not create Hamm before Babe completes and strategy is valid
- Does not create Pumba before Hamm content is valid
- Produces a `strategy` artifact before `SYNTHESIS_READY`
- Creates complete phase-specific subtask descriptions
- Does not create title-only subtasks
- Validates every artifact before advancing
- Posts final summary only after final validation

### Babe

- Executes the assigned research task directly
- Does not spend the run on broad Paperclip reference discovery
- Produces `ARTIFACT: type: audit`
- Includes all required audit sections
- Includes `critical_flags`, even if empty
- Posts artifact on the parent issue
- Emits `AUDIT_READY` on the parent issue

### Hamm

- Starts only after Porky explicitly delegates content
- Consumes Porky’s strategy artifact
- Respects `excluded_claims` and `safe_framing` if present
- Produces `ARTIFACT: type: content_set`
- Produces exactly 3 content items
- Posts artifact on the parent issue
- Emits `CONTENT_SET_READY` on the parent issue

### Pumba

- Starts only after Porky explicitly delegates Notion + Delivery
- Does not run early
- Does not emit noisy `BLOCKED` before explicit delegation
- Persists content into Notion Content Pipeline
- Persists research into Research Babe
- Does not store final content in `Notes`
- Preserves thread tweets as ordered body blocks
- Sends Telegram summary once
- Posts final artifacts and signals on the parent issue

---

## Test 2 — Weekly 7-day test

Use this after the controlled 3-day test passes.

### Goal

Validate a normal weekly content workflow using a realistic high-level prompt.

### Expected output

- 7 X posts or threads, one per day
- 2 LinkedIn posts during the week
- 9 total content items
- Research Babe page
- Telegram summary

### What it validates

- Normal weekly distribution
- Multi-platform mix
- Week field generation
- Notion deduplication
- Research traceability
- Porky’s ability to expand a vague operator prompt
- Rich subtask descriptions under a production-like request

---

## Weekly 7-day test issue

Copy this into a new parent issue assigned to Porky:

```text
TITLE: Social Media Content Creation — Week #2 May

DESCRIPTION:

Generate content for the Piggy Wallet X and LinkedIn accounts for the week of May 11 to May 17, 2026.

I want:

- 1 post for X per day
- 2 posts for LinkedIn during the week

Take into account that last week the Piggy Wallet team attended Consensus Miami.

Please research relevant speakers, side events, activities, and themes from Consensus Miami, and use that context to create useful content for Piggy Wallet’s audience.

The content should stay aligned with Piggy Wallet’s mission:

- helping families in emerging markets protect savings from inflation
- making financial education easier for families
- using Web3 infrastructure under the hood while keeping the experience simple
- building long-term trust through educational content

Please complete the full workflow:

Research → Strategy → Content → Notion → Delivery
```

This issue is intentionally high-level.

It should work without listing every internal signal, artifact, or subtask template.

---

## Weekly 7-day pass/fail criteria

The weekly test passes only if:

- Porky creates only Babe first
- Babe subtask has a complete description
- Babe completes the audit on the parent issue
- Porky validates audit before creating strategy
- Porky creates Hamm only after strategy is valid
- Hamm subtask has a complete description
- Hamm creates 9 total content items
- Hamm posts content_set on the parent issue
- Porky creates Pumba only after content_set validation
- Pumba subtask has a complete description
- Pumba persists 9 content items in Notion
- Pumba persists Research Babe
- Pumba sends Telegram once
- All final artifacts and signals are visible on the parent issue

The weekly test fails if:

- Hamm is created before Babe completes
- Pumba is created before Hamm completes
- Any subtask is title-only
- Any final artifact exists only on a subtask
- Content is persisted in Notes
- Research Babe is missing
- Telegram is duplicated
- Porky completes without final validation

---

## Test 3 — Extended 11-day test

Use this for larger campaign validation.

### Goal

Validate a longer content range and stronger scheduling behavior.

### Expected output

- 11 total content items, unless otherwise specified
- Exactly 1 item per scheduled date when requested
- At least 3 X threads per week when weekly thread rules apply
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
- Stability of the orchestration loop across a larger workload

---

## Validation checklist

After every E2E test, verify the following manually or automatically.

### Parent issue

- Parent issue contains all final artifacts
- Parent issue contains all final signals
- Parent issue shows clear phase progression
- Final Porky summary exists
- No final completion depends only on subtask-only evidence

### Subtasks

- Babe subtask exists first
- Babe subtask has a complete description
- Hamm subtask is created only after audit and strategy validation
- Hamm subtask has a complete description
- Pumba subtask is created only after content validation
- Pumba subtask has a complete description
- No title-only subtasks exist

### Babe

- `ARTIFACT: type: audit` exists
- Artifact appears before `AUDIT_READY`
- Required sections are present
- `critical_flags` exists, even if empty
- Uncertainty is included
- Recommended content angles are included
- Artifact and signal appear on parent issue
- Babe does not run broad Paperclip reference discovery before completing audit

### Porky

- `ARTIFACT: type: strategy` exists
- Artifact appears before `SYNTHESIS_READY`
- Date range is present
- Expected item count is present
- Messaging pillars are present
- Content angles are present
- Strategy maps to audit
- If risks exist, `excluded_claims` and `safe_framing` exist
- Porky does not advance with invalid artifacts
- Porky does not create downstream subtasks early

### Hamm

- `ARTIFACT: type: content_set` exists
- Artifact appears before `CONTENT_SET_READY`
- `item_count` matches actual items
- Every content item has required fields
- Posts include `body`
- Threads include `thread_id` and `tweets`
- Thread tweets are ordered
- No duplicate titles
- Content respects `excluded_claims`
- Content uses `safe_framing` when required
- Artifact and signal appear on parent issue

### Pumba

- Pumba starts only after explicit delegation
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
- `notion_content_pipeline` artifact appears on parent issue
- `telegram_delivery` artifact appears on parent issue
- Both Pumba signals appear on parent issue

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

### Failure test 6 — Title-only subtask

Simulate:

- Porky creates a Babe, Hamm, or Pumba subtask with only a title and no complete description

Expected behavior:

- The delegation is invalid
- Porky must update the subtask description if possible
- If Porky cannot update it, Porky must block and identify missing context
- The assigned specialist should not guess missing requirements

---

### Failure test 7 — Premature Hamm execution

Simulate:

- Hamm starts before Babe audit and Porky strategy are valid

Expected behavior:

- Hamm must not generate content
- Hamm should block or stop based on missing strategy
- Porky should treat the premature execution as invalid
- Pumba must not run

---

### Failure test 8 — Premature Pumba execution

Simulate:

- Pumba wakes before Porky explicitly delegates Notion + Delivery

Expected behavior:

- Pumba must not persist content
- Pumba must not send Telegram
- Pumba must not emit `NOTION_SYNC_COMPLETE`
- Pumba must not emit `DELIVERY_COMPLETE`
- Pumba should exit cleanly or leave a waiting note
- This should not be treated as workflow failure unless Porky had already delegated Pumba

---

### Failure test 9 — Missing critical_flags

Simulate:

- Babe publishes an audit artifact without `critical_flags`

Expected behavior:

- Porky treats the audit artifact as incomplete
- Porky blocks before strategy
- Babe must republish the audit artifact with `critical_flags`, even if empty

---

### Failure test 10 — Unsafe claim used in content

Simulate:

- Babe flags an unsafe claim
- Porky strategy excludes the claim
- Hamm uses the excluded claim anyway

Expected behavior:

- Porky rejects `CONTENT_SET_READY`
- Workflow blocks
- Hamm must revise content to remove excluded claims

---

## Regression testing

Run a controlled E2E test after changing:

- Any `AGENTS.md`
- Any `HEARTBEAT.md`
- Signal contract
- Artifact contract
- Notion schema
- Pumba persistence logic
- Porky validation gates
- Delivery behavior
- Subtask creation behavior
- Direct assignment execution behavior

Recommended regression test:

- 3-day controlled E2E
- Vague parent issue
- 1 X thread
- 1 X post
- 1 LinkedIn post
- Research Babe required
- Telegram required

---

## Passing criteria

An E2E test passes only when:

- All required artifacts exist
- All required signals exist
- Artifacts appear before completion signals
- All final artifacts and signals appear on the parent issue
- Porky validates each phase
- Porky delegates sequentially
- Porky creates complete subtask descriptions
- No title-only subtasks exist
- Hamm does not start early
- Pumba does not start early
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

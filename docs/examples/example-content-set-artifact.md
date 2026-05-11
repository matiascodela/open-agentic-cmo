# Example Content Set Artifact

This example shows what a valid `content_set` artifact from Hamm should look like.

The `content_set` artifact is the structured, publication-ready content output used by Porky for validation and by Pumba for Notion persistence.

Hamm must publish this artifact before emitting `CONTENT_SET_READY`.

The artifact and signal must be posted on the original parent issue.

Subtask-only output is not enough for workflow completion.

---

## Purpose

The content set artifact turns Porky’s strategy into executable content.

It should include:

- Full content items
- Required metadata
- Scheduled dates
- Platform information
- Content type
- Version
- Week
- Tags and mentions
- Post body text
- Thread tweet arrays
- Strategy alignment
- Safe handling of excluded claims and uncertain topics

A summary is not enough.

Hamm must include the full content for every item.

Pumba should be able to persist the artifact into Notion without guessing.

---

## Required behavior

Hamm must only generate content after Porky explicitly delegates the Content phase.

Hamm must not start because a future subtask exists.

Hamm must not start before:

- Babe audit artifact exists
- `AUDIT_READY` exists or has been reconciled
- Porky strategy artifact exists
- `SYNTHESIS_READY` exists or has been reconciled
- Porky explicitly delegated the content phase to Hamm

If those conditions are not met, Hamm must not generate content.

---

## Strategy compliance

Hamm must strictly follow Porky’s strategy artifact.

Every content item should map to:

- a messaging pillar
- a content angle
- a scheduled date
- a platform
- a language
- a content format
- a strategic goal

Hamm must follow:

- `brand_constraints`
- `content_rules`
- `weekly_structure`
- `expected_items`
- `excluded_claims`, if present
- `safe_framing`, if present

If the strategy artifact includes `excluded_claims`, Hamm must not use those claims anywhere.

Excluded claims are forbidden in:

- titles
- hooks
- posts
- threads
- CTAs
- hashtags
- tags
- summaries
- platform copy
- implied framing

If the strategy artifact includes `safe_framing`, Hamm must use that framing when writing around uncertain topics.

---

## Required content item fields

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

Allowed platforms:

```text
X
LinkedIn
```

Allowed languages:

```text
EN
ES
bilingual
```

Allowed content types:

```text
post
thread
```

Version rule:

```text
version: v1
```

unless Porky explicitly specifies another version.

Week rule:

```text
week: YYYY-WXX
```

Example:

```text
2026-05-11 → 2026-W20
```

If `week` is missing, the artifact is invalid.

---

## Thread requirements

For threads, include:

```text
thread_id:
tweets:
```

Thread ID format:

```text
TH-YYYY-MM-DD-N
```

Example:

```text
TH-2026-05-11-1
```

Each thread must be unique.

Tweets must be:

- an ordered array of strings
- 4–7 tweets
- non-empty
- non-duplicated
- logically progressive
- publication-ready
- ready to become ordered Notion body blocks

Threads must not be one merged paragraph.

Threads must not be stored as a summary.

Threads must not require Pumba to infer ordering.

---

## Example

```text
ARTIFACT:
type: content_set
issue: PIG-101
subtask_issue: PIG-103
item_count: 3
content_items:

- title: Why a piggy bank is no longer enough
  platform: X
  language: EN
  content_type: thread
  scheduled_date: 2026-05-11
  vertical: Inflation Protection
  version: v1
  week: 2026-W20
  tags_and_mentions:
    - #FinancialEducation
    - #FamilyFinance
  thread_id: TH-2026-05-11-1
  tweets:
    - A piggy bank teaches kids to save. But in inflationary economies, saving alone is no longer enough.
    - When prices rise, money that sits still can quietly lose purchasing power.
    - That means families need two things: better money habits and better protection systems.
    - Piggy Wallet is designed around that idea: helping families build saving habits while making modern financial tools easier to understand.
    - The future of financial education is not more complexity. It is simpler defaults that families can actually use.

- title: Money habits start before kids earn money
  platform: X
  language: EN
  content_type: post
  scheduled_date: 2026-05-12
  vertical: Financial Education
  version: v1
  week: 2026-W20
  tags_and_mentions:
    - #MoneyHabits
    - #FinancialLiteracy
  body: Kids do not need to earn money before they start learning about it. Small habits — saving, waiting, choosing, planning — shape how they understand value later. Financial education starts long before the first paycheck.

- title: Families need simpler financial defaults
  platform: LinkedIn
  language: EN
  content_type: post
  scheduled_date: 2026-05-13
  vertical: Trust & Simplicity
  version: v1
  week: 2026-W20
  tags_and_mentions:
    - #Fintech
    - #FinancialEducation
    - #FamilyFinance
  body: Parents do not need more financial complexity. They need tools that make better money habits easier to build and safer to maintain. At Piggy Wallet, we believe family finance should feel simple, educational, and protective — especially in markets where inflation can quietly erode savings over time.
```

---

## Example with excluded claims and safe framing

If Porky’s strategy includes:

```text
excluded_claims:
- claim: Piggy Wallet guarantees inflation protection.
  reason: Guarantee language is unsupported and unsafe.

- claim: Piggy Wallet provides risk-free savings.
  reason: Risk-free financial claims are unsupported and unsafe.

safe_framing:
- framing: Piggy Wallet is designed to help families build better savings habits.
  reason: This keeps the claim educational and mission-aligned without implying guaranteed outcomes.

- framing: Piggy Wallet uses modern infrastructure under the hood while keeping the experience simple.
  reason: This explains the technology posture without requiring users to understand crypto.
```

Then Hamm’s content must avoid the excluded claims and use the safe framing.

Valid example:

```text
body: In inflationary environments, families need habits and tools that help them think more intentionally about saving. Piggy Wallet is designed to support that journey with simple financial education and modern infrastructure under the hood.
```

Invalid example:

```text
body: Piggy Wallet guarantees inflation protection and gives families risk-free savings through crypto.
```

Why invalid:

- Uses excluded claims.
- Makes unsupported financial guarantees.
- Adds unsafe crypto framing.
- Violates Porky’s strategy constraints.

---

## Matching signal

After publishing the full artifact on the parent issue, Hamm may emit:

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

The signal must not be emitted before the artifact is visible.

Hamm may also copy the same artifact and signal to the subtask for traceability.

The parent issue is mandatory.

---

## Validation checklist

A valid content set artifact should include:

- [ ] `ARTIFACT:` header
- [ ] `type: content_set`
- [ ] Parent issue ID
- [ ] Hamm subtask issue ID
- [ ] `item_count`
- [ ] `content_items`
- [ ] `item_count` matches the number of listed items
- [ ] Every item has `title`
- [ ] Every item has `platform`
- [ ] Every item has `language`
- [ ] Every item has `content_type`
- [ ] Every item has `scheduled_date`
- [ ] Every item has `vertical`
- [ ] Every item has `version`
- [ ] Every item has `week`
- [ ] Every item has `tags_and_mentions`
- [ ] Every post has `body`
- [ ] Every thread has `thread_id`
- [ ] Every thread has `tweets`
- [ ] Thread tweets are ordered
- [ ] Thread tweets are not duplicated
- [ ] Threads include 4–7 tweets
- [ ] No duplicate titles
- [ ] No duplicate scheduled dates unless explicitly allowed
- [ ] Content follows Porky’s strategy artifact
- [ ] Content maps to strategy pillars and angles
- [ ] Content respects `brand_constraints`
- [ ] Content follows `content_rules`
- [ ] Content avoids `excluded_claims`
- [ ] Content uses `safe_framing` when required
- [ ] Content avoids unsupported product, financial, chain, or Web3 claims
- [ ] Artifact is posted on the parent issue
- [ ] Signal emitted only after artifact is visible
- [ ] `CONTENT_SET_READY` is posted on the parent issue

---

## Common invalid examples

Invalid content set artifacts include:

- A summary without full content
- “Generated 3 posts” without listing the posts
- Missing post body
- Missing thread tweets
- Thread stored as a single paragraph instead of a tweet array
- Missing `scheduled_date`
- Missing `version`
- Missing `week`
- Missing `content_type`
- Missing `tags_and_mentions`
- Duplicate titles
- Duplicate angles
- Invalid platform names
- Content that does not map to Porky’s strategy
- Content that uses excluded claims
- Content that invents unsupported product capabilities
- Content that makes unsupported financial guarantees
- Artifact posted only on the subtask and not the parent issue
- `CONTENT_SET_READY` emitted before the artifact is visible

---

## Invalid example — summary instead of content

```text
ARTIFACT:
type: content_set
issue: PIG-101
subtask_issue: PIG-103
item_count: 3

summary:
Generated 1 X thread, 1 X post, and 1 LinkedIn post for May 11–13, 2026.
```

Why invalid:

- No `content_items`
- No full post body
- No thread tweets
- No scheduled item metadata
- Pumba cannot persist this into Notion without guessing

---

## Invalid example — missing required metadata

```text
ARTIFACT:
type: content_set
issue: PIG-101
subtask_issue: PIG-103
item_count: 1
content_items:

- title: Money habits start early
  platform: X
  content_type: post
  body: Financial education starts before the first paycheck.
```

Why invalid:

- Missing `language`
- Missing `scheduled_date`
- Missing `vertical`
- Missing `version`
- Missing `week`
- Missing `tags_and_mentions`

---

## Invalid example — thread as one paragraph

```text
ARTIFACT:
type: content_set
issue: PIG-101
subtask_issue: PIG-103
item_count: 1
content_items:

- title: Why a piggy bank is no longer enough
  platform: X
  language: EN
  content_type: thread
  scheduled_date: 2026-05-11
  vertical: Inflation Protection
  version: v1
  week: 2026-W20
  tags_and_mentions:
    - #FinancialEducation
  thread_id: TH-2026-05-11-1
  body: A piggy bank teaches kids to save. But inflation changes the lesson. Families need better habits and safer systems. Piggy Wallet helps make this simpler.
```

Why invalid:

- `content_type` is `thread`, but there is no `tweets` array.
- Pumba cannot preserve tweet order as Notion body blocks.
- The thread is not structurally persistable.

---

## Invalid example — excluded claim used

Assume Porky’s strategy includes:

```text
excluded_claims:
- claim: Piggy Wallet guarantees inflation protection.
  reason: Unsupported financial guarantee.
```

Invalid content:

```text
ARTIFACT:
type: content_set
issue: PIG-101
subtask_issue: PIG-103
item_count: 1
content_items:

- title: Guaranteed protection for family savings
  platform: X
  language: EN
  content_type: post
  scheduled_date: 2026-05-12
  vertical: Inflation Protection
  version: v1
  week: 2026-W20
  tags_and_mentions:
    - #FamilyFinance
  body: Piggy Wallet guarantees inflation protection for family savings, making it a risk-free way to save for your child’s future.
```

Why invalid:

- Uses an excluded claim.
- Makes unsupported guarantee language.
- Uses “risk-free” financial language.
- Violates Piggy Wallet safety rules.
- Porky must reject this before Pumba runs.

---

## Invalid example — unsupported chain claim

Invalid content:

```text
body: Piggy Wallet is powered by the fastest blockchain, giving families the best chain for savings.
```

Why invalid:

- Invents or implies unverified technical positioning.
- Uses chain superiority language.
- Makes unsupported product infrastructure claims.
- Violates Web3 safety rules unless explicitly validated by Porky’s strategy.

Better:

```text
body: Piggy Wallet uses modern infrastructure under the hood while keeping the experience simple for families.
```

---

## Notes

The content set artifact is what Pumba uses to persist content into Notion.

For this reason, every content item must be complete and unambiguous.

Pumba should not need to infer:

- where content goes
- which platform it belongs to
- which date it is scheduled for
- whether it is a post or thread
- what week it belongs to
- what version it is
- whether a claim is safe
- how to split a thread into blocks

If Pumba has to guess, the artifact is invalid.

If Hamm has to guess what is strategically safe, the strategy artifact is incomplete.

If Porky has to guess whether the content used an excluded claim, the content_set artifact is unsafe.

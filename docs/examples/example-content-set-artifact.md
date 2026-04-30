# Example Content Set Artifact

This example shows what a valid `content_set` artifact from Hamm should look like.

The `content_set` artifact is the structured, publication-ready content output used by Porky for validation and by Pumba for Notion persistence.

Hamm must publish this artifact before emitting `CONTENT_SET_READY`.

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

A summary is not enough.

Hamm must include the full content for every item.

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
    - When prices rise, money that sits still quietly loses purchasing power.
    - That means families need two things: better money habits and better protection systems.
    - Piggy Wallet is built around that idea: helping families teach saving while protecting value in a simple way.
    - The future of financial education is not more complexity. It is safer defaults that families can actually understand.

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

## Matching signal

After publishing the full artifact, Hamm may emit:

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
- [ ] Signal emitted only after artifact is visible

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
- Duplicate titles
- Invalid platform names
- `CONTENT_SET_READY` emitted before the artifact is visible

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

If Pumba has to guess, the artifact is invalid.

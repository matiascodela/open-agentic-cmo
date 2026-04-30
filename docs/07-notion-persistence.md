# Notion Persistence

Notion is the human-reviewable workspace for Open Agentic CMO.

Pumba is responsible for persisting validated workflow artifacts into Notion after Porky has validated that the required upstream artifacts exist.

Notion persistence has two main outputs:

1. Content Pipeline entries
2. Research Babe page

Both are required before `NOTION_SYNC_COMPLETE` can be considered valid.

---

## Why Notion persistence matters

The goal of Open Agentic CMO is not only to generate content.

The goal is to produce structured, reviewable, and operationally useful outputs.

Notion persistence allows humans to:

- Review final content
- Approve or edit posts
- See scheduled dates
- Track platforms and formats
- Understand the research context behind the campaign
- Verify what the agents produced
- Maintain a content operating system

Without correct Notion persistence, the workflow is incomplete.

---

## Owner

Notion persistence is owned by Pumba.

Pumba is responsible for:

- Validating the `content_set` artifact
- Validating the `audit` artifact
- Creating or updating Content Pipeline pages
- Creating or updating the Research Babe page
- Preserving structured metadata
- Preserving post and thread body content
- Preventing duplicates
- Verifying Notion state before emitting `NOTION_SYNC_COMPLETE`

Pumba must not create content, define strategy, perform research, or invent missing fields.

---

## Required inputs

Pumba requires the following inputs before starting Notion persistence:

- Valid `CONTENT_SET_READY` signal
- Valid `content_set` artifact from Hamm
- Valid `AUDIT_READY` signal
- Valid `audit` artifact from Babe
- Parent issue ID
- Pumba subtask issue ID

If any required input is missing, Pumba must emit `BLOCKED`.

---

## Required outputs

Pumba must persist both:

1. All content items into the Notion Content Pipeline
2. Babe’s research artifact into a dedicated Notion page called `Research Babe`

Pumba may emit `NOTION_SYNC_COMPLETE` only after both outputs exist and pass verification.

---

## Content Pipeline

The Content Pipeline stores final content items.

Each content item becomes one Notion page.

Content items may include:

- X posts
- X threads
- LinkedIn posts

---

## Content Pipeline required properties

Each content page should include the following properties:

| Property | Required | Source |
|---|---:|---|
| Title | Yes | `content_item.title` |
| Platform | Yes | `content_item.platform` |
| Status | Yes | `Scheduled` |
| Scheduled Date | Yes | `content_item.scheduled_date` |
| Vertical | Yes | `content_item.vertical` |
| Version | Yes | `content_item.version` |
| Week | Yes | `content_item.week` |
| Content Type | Yes | `content_item.content_type` |
| Thread ID | For threads | `content_item.thread_id` |
| Thread Order | Optional | item order / thread ordering |

Do not leave required properties empty.

If the Notion schema does not support one of these properties, Pumba should use the available schema safely and block if the missing property is required by the workflow.

---

## Body, not Notes

This is one of the most important persistence rules.

Content must be written into the Notion page body.

Content must not be stored in:

- `Notes`
- metadata fields
- auxiliary text fields
- fallback text fields

The `Notes` property should remain empty unless a future contract explicitly requires it.

---

## Post body persistence

For content items with:

`content_type: post`

Pumba should create one paragraph block in the Notion page body containing the full post body.

The post body must come from:

`content_item.body`

Pumba must not:

- Put the post body into `Notes`
- Split the body unnecessarily
- Add unsupported metadata into the body
- Rewrite the post creatively
- Invent missing body text

---

## Thread body persistence

For content items with:

`content_type: thread`

Pumba should create multiple paragraph blocks in the Notion page body.

Each tweet should become one block.

The tweet order must be preserved exactly.

Example:

Tweet 1 text

Tweet 2 text

Tweet 3 text

Tweet 4 text

Pumba must not:

- Merge all tweets into one block
- Store the thread in `Notes`
- Lose tweet ordering
- Add numbering prefixes unless Hamm already provided them
- Rewrite the thread creatively
- Invent missing tweets

---

## Thread formatting rule

Pumba should preserve tweets as clean text.

Pumba must not add:

- `1/`
- `2/`
- `Tweet 1:`
- `Tweet 2:`
- numbering prefixes

The only exception is when Hamm already provided numbering as part of the artifact.

---

## Research Babe

Research Babe is a dedicated Notion page that stores Babe’s research artifact.

Its purpose is to preserve the research and analysis behind the content.

This allows humans to understand:

- What Babe researched
- What signals were processed
- What context informed the strategy
- Why the content angles exist
- What uncertainty remains

---

## Research Babe page title

The Research Babe page should be titled:

`Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>`

If date range is unavailable:

`Research Babe — <PARENT_ISSUE_ID>`

Examples:

`Research Babe — PIG-77 — May 11–13, 2026`

`Research Babe — PIG-77`

---

## Research Babe page body

The Research Babe page body must include:

1. Research Summary
2. Key Signals
3. Inconsistencies
4. Uncertainty / Data Gaps
5. Opportunities
6. Recommended Content Angles
7. Source / Workflow Metadata

Use clear headings.

The page should be readable by humans.

---

## Research Babe properties

If the Notion database or page supports properties, set:

| Property | Value |
|---|---|
| Title | `Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>` |
| Issue | `<PARENT_ISSUE_ID>` |
| Type | `Research` |
| Agent | `Babe` |
| Status | `Ready for Review` |
| Date Range | `<DATE_RANGE>` |
| Week | `<WEEK>` if available |
| Source Artifact | `audit` |

If some properties do not exist in Notion, do not invent unsupported fields.

Use the available schema safely.

---

## What not to do with research

Pumba must not:

- Store Babe research inside individual content entries
- Mix research into post or thread pages
- Overwrite content pipeline pages with research
- Skip research persistence if the audit artifact exists
- Create multiple Research Babe pages for the same parent issue
- Invent missing research sections

---

## Deduplication

Pumba must prevent duplicates.

### Content deduplication key

Use:

`parent_issue_id + scheduled_date + platform + title`

If a matching content page exists:

- update it

If it does not exist:

- create it

Never create duplicate content entries.

---

### Research Babe deduplication key

Use:

`parent_issue_id + babe_research`

If a matching Research Babe page exists:

- update it

If it does not exist:

- create it

Never create duplicate Research Babe pages for the same parent issue.

---

## Date handling

Pumba must use the `scheduled_date` exactly as provided by Hamm.

Pumba must never use:

- current date
- fallback date
- issue creation date
- task date
- execution timestamp

If `scheduled_date` is missing or invalid, Pumba must block.

---

## Required verification before NOTION_SYNC_COMPLETE

Before emitting `NOTION_SYNC_COMPLETE`, Pumba must verify the Notion state.

### Content Pipeline verification

Check that:

- all content items exist
- no duplicate content entries exist
- scheduled dates are correct
- `Version` is populated
- `Week` is populated
- `Content Type` is populated
- `Status` is `Scheduled`
- page body exists
- content is not stored in `Notes`
- posts have body content
- threads are split into ordered body blocks

---

### Research Babe verification

Check that:

- Research Babe page exists
- audit artifact is persisted
- required sections are present
- page is readable for human review
- no duplicate Research Babe page exists

---

## NOTION_SYNC_COMPLETE meaning

`NOTION_SYNC_COMPLETE` means both:

1. Content Pipeline entries are persisted and verified.
2. Research Babe page is persisted and verified.

Anything less is incomplete.

Pumba must not emit `NOTION_SYNC_COMPLETE` if:

- content pages exist but Research Babe is missing
- Research Babe exists but content pages are missing
- content is stored in `Notes`
- required fields are empty
- duplicate pages exist
- verification fails

---

## Failure conditions

Pumba must emit `BLOCKED` if:

- `CONTENT_SET_READY` is missing
- `content_set` artifact is missing
- `AUDIT_READY` is missing
- `audit` artifact is missing
- required content fields are missing
- scheduled date is invalid
- Notion persistence fails
- content is stored in `Notes`
- Research Babe page cannot be created or updated
- duplicates cannot be resolved
- verification fails

---

## Reconciliation behavior

If Content Pipeline entries already exist:

1. Validate correctness.
2. If correct, do not rewrite.
3. If incorrect, update only incorrect fields or blocks.

If Research Babe page already exists:

1. Validate correctness.
2. If correct, do not rewrite.
3. If incomplete, update missing sections.

If partial persistence exists:

1. Resume only missing steps.
2. Do not recreate valid pages.
3. Do not overwrite correct data unnecessarily.

---

## Human review model

The Notion workspace is designed for human review.

Humans should be able to inspect:

- what content was created
- when it is scheduled
- which platform it belongs to
- whether it is a post or thread
- which version it is
- which week it belongs to
- what research informed the content
- what uncertainty or gaps remain

This is why both Content Pipeline and Research Babe are required.

---

## Summary

Notion persistence is not just storage.

It is the operational interface for the content system.

Correct persistence makes the workflow:

- reviewable
- auditable
- schedulable
- traceable
- safer to operate

Pumba’s job is not to “save content.”

Pumba’s job is to guarantee that structured artifacts become clean, complete, human-reviewable Notion outputs.

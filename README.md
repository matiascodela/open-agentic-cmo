# Open Agentic CMO

**Open Agentic CMO** is an open-source deterministic multi-agent CMO system for running a complete content marketing workflow:

Research → Strategy → Content → Notion → Delivery

It was originally built for **Piggy Wallet**, a fintech + edtech product helping families in emerging markets protect savings from inflation and build healthier financial habits.

The system is designed to behave like an autonomous CMO team: it researches, synthesizes strategy, generates platform-native content, persists the output into Notion, and sends a delivery summary — all through structured agents, signals, artifacts, and validation gates.

---

## Why this exists

Most AI agent workflows fail because they rely on loose text, implicit assumptions, and “looks good” reasoning.

**Open Agentic CMO** takes a different approach.

It is built around:

- **Signals** — explicit workflow transition markers
- **Artifacts** — structured outputs that can be validated
- **Validation gates** — hard checks before moving to the next phase
- **Idempotency** — no duplicate work, no duplicate Notion entries, no duplicate delivery
- **Human reviewability** — both content and research context are persisted for review

The goal is not just to generate content.

The goal is to build a reliable, auditable, autonomous marketing operating system.

---

## Core workflow

Research → Strategy → Content → Notion → Delivery

### 1. Research

Babe analyzes available signals and produces a structured research artifact.

### 2. Strategy

Porky synthesizes the research into a structured strategy artifact.

### 3. Content

Hamm transforms the strategy into a structured content set.

### 4. Notion

Pumba persists:

1. Final content into the Notion Content Pipeline
2. Babe’s research into a dedicated Notion page called `Research Babe`

### 5. Delivery

Pumba sends a Telegram summary and emits final delivery completion.

---

## Agents

### Porky — Orchestrator / CEO-CMO

Porky owns the full workflow.

Responsibilities:

- Orchestration
- Signal scanning
- State reconciliation
- Strategy synthesis
- Artifact validation
- Validation gates
- Idempotency
- Blocking invalid workflows

Porky does **not** create content, perform research, or persist to Notion.

### Babe — Research Agent

Babe turns raw signals into structured research artifacts.

Responsibilities:

- Audience research
- Digital presence audit
- Trend analysis
- Competitive signals
- Opportunity identification
- Recommended content angles

Babe emits: `AUDIT_READY`

### Hamm — Content Lead

Hamm turns strategy into structured, publication-ready content.

Responsibilities:

- X posts
- X threads
- LinkedIn posts
- Content formatting
- Platform-native writing
- Content artifact generation

Hamm emits: `CONTENT_SET_READY`

### Pumba — Notion + Delivery Agent

Pumba turns structured artifacts into persisted and delivered outputs.

Responsibilities:

- Persist content into Notion
- Persist Babe research into `Research Babe`
- Verify Notion structure
- Avoid duplicates
- Send Telegram summary

Pumba emits:

- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

---

## System design principles

### Signal-driven

Every phase transition requires a valid signal.

Signal format:

SIGNAL:  
type: `<SIGNAL_TYPE>`  
producer: `<AGENT_NAME>`  
issue: `<PARENT_ISSUE_ID>`  
subtask_issue: `<SUBTASK_ISSUE_ID>`  
status: `<COMPLETE | FAILED>`  
artifact: `<ARTIFACT_NAME>`  
timestamp: `<TIMESTAMP>`

### Artifact-driven

Signals alone are not enough.

Each signal must correspond to a valid structured artifact.

Example artifact header:

ARTIFACT:  
type: `content_set`  
issue: `<PARENT_ISSUE_ID>`  
subtask_issue: `<HAMM_SUBTASK_ISSUE_ID>`  
item_count: `3`  
content_items:

### Validation-first

The workflow never advances with missing, incomplete, or invalid artifacts.

If a phase fails validation, Porky blocks the workflow and requests a correction.

### Idempotent

The system is designed to avoid:

- Duplicate content generation
- Duplicate Notion pages
- Duplicate `Research Babe` pages
- Duplicate Telegram summaries
- Duplicate workflow signals

### Human-reviewable

The system does not only persist final content.

It also persists the research context behind the content so humans can understand the reasoning behind each campaign.

---

## Artifact types

The core artifact types are:

- `audit`
- `strategy`
- `content_set`
- `notion_content_pipeline`
- `telegram_delivery`

---

## Audit artifact

Produced by Babe.

Required sections:

- `summary`
- `key_signals`
- `inconsistencies`
- `uncertainty`
- `opportunities`
- `recommended_content_angles`

Example:

ARTIFACT:  
type: `audit`  
issue: `<PARENT_ISSUE_ID>`  
subtask_issue: `<BABE_SUBTASK_ISSUE_ID>`  
entity: `Piggy Wallet`  
timestamp:  
sections:

summary:

key_signals:

inconsistencies:

uncertainty:

opportunities:

recommended_content_angles:

---

## Strategy artifact

Produced by Porky.

Required sections:

- `date_range`
- `expected_items`
- `messaging_pillars`
- `content_angles`
- `weekly_structure`
- `brand_constraints`
- `content_rules`

Example:

ARTIFACT:  
type: `strategy`  
issue: `<PARENT_ISSUE_ID>`  
date_range:  
expected_items:  
messaging_pillars:  
content_angles:  
weekly_structure:  
brand_constraints:  
content_rules:

---

## Content set artifact

Produced by Hamm.

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

Example:

ARTIFACT:  
type: `content_set`  
issue: `<PARENT_ISSUE_ID>`  
subtask_issue: `<HAMM_SUBTASK_ISSUE_ID>`  
item_count: `3`  
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
  content_type: `thread`  
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

---

## Signal contract

Allowed signals:

- `AUDIT_READY`
- `SYNTHESIS_READY`
- `CONTENT_SET_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`
- `BLOCKED`
- `RETRY_REQUIRED`
- `TIMEOUT_DETECTED`
- `RECOVERY_IN_PROGRESS`
- `RECOVERY_COMPLETE`

Signals must be explicit, structured, and valid.

Inline or malformed signals should not be used for phase transitions.

---

## Notion persistence model

Pumba persists two types of Notion outputs.

### 1. Content Pipeline

Each content item becomes a Notion page.

Required properties:

- `Title`
- `Platform`
- `Status`
- `Scheduled Date`
- `Vertical`
- `Version`
- `Week`
- `Content Type`

For threads:

- `Thread ID`

The content itself must be written into the page body.

Content must **not** be stored in `Notes`.

### 2. Research Babe

For every workflow, Pumba creates or updates a page titled:

`Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>`

This page includes:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Source / Workflow Metadata

This allows humans to review not just the final content, but also the research and analysis behind it.

---

## Example workflow

A controlled three-day test might produce:

Date range: May 11–13, 2026

Expected output:

- 1 X thread
- 1 X post
- 1 LinkedIn post
- 1 Research Babe page
- 1 Telegram summary

Expected phase sequence:

1. Babe → `AUDIT_READY`
2. Porky → `SYNTHESIS_READY`
3. Hamm → `CONTENT_SET_READY`
4. Pumba → `NOTION_SYNC_COMPLETE`
5. Pumba → `DELIVERY_COMPLETE`

---

## Repository structure

Recommended structure:

open-agentic-cmo/  
├── README.md  
├── LICENSE  
├── .env.example  
├── agents/  
│   ├── porky/  
│   │   └── AGENTS.md  
│   ├── babe/  
│   │   └── AGENTS.md  
│   ├── hamm/  
│   │   └── AGENTS.md  
│   └── pumba/  
│       └── AGENTS.md  
├── docs/  
│   ├── 01-overview.md  
│   ├── 02-architecture.md  
│   ├── 03-agent-contracts.md  
│   ├── 04-signal-contract.md  
│   ├── 05-artifact-contracts.md  
│   ├── 06-orchestration-loop.md  
│   ├── 07-notion-persistence.md  
│   ├── 08-testing-e2e.md  
│   └── 09-roadmap.md  
├── examples/  
│   ├── example-audit-artifact.md  
│   ├── example-strategy-artifact.md  
│   ├── example-content-set-artifact.md  
│   ├── example-signals.md  
│   └── example-controlled-e2e-test.md  
├── schemas/  
│   ├── audit.schema.json  
│   ├── strategy.schema.json  
│   ├── content-set.schema.json  
│   └── signal.schema.json  
└── tests/  
    ├── controlled-e2e-test.md  
    ├── validation-checklist.md  
    └── failure-cases.md

---

## Environment variables

Do not commit real API keys, tokens, database IDs, or private URLs.

Use `.env.example` for placeholders:

NOTION_API_KEY=your_notion_api_key_here  
NOTION_CONTENT_DATABASE_ID=your_notion_content_database_id_here  
NOTION_RESEARCH_DATABASE_ID=your_notion_research_database_id_here  
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here  
TELEGRAM_CHAT_ID=your_telegram_chat_id_here

---

## Current status

The current system supports:

- Structured research artifacts
- Structured strategy artifacts
- Structured content artifacts
- X posts
- X threads
- LinkedIn posts
- Notion Content Pipeline persistence
- Research Babe Notion page persistence
- Telegram delivery summaries
- Signal-based orchestration
- Validation gates
- Idempotency rules

Piglet, the image generation agent, is currently excluded from the core workflow to preserve system stability.

---

## Roadmap

Planned improvements:

- JSON schemas for all artifacts
- Automated validation scripts
- Notion setup guide
- Installation/setup guide
- Example workflows
- Reusable issue templates
- Content quality scoring
- Analytics feedback loop
- Optional image generation module
- Multi-provider fallback support
- Generalized support for other brands and teams

---

## Who this is for

Open Agentic CMO is useful for:

- Founders
- Growth teams
- Content teams
- AI agent builders
- Startup marketers
- Notion-based content operations teams
- Teams experimenting with autonomous marketing workflows

---

## Philosophy

This project is based on a simple idea:

> Agents should not just write.  
> Agents should produce structured, validated, traceable work.

Open Agentic CMO treats marketing as an operating system:

Research becomes strategy.  
Strategy becomes content.  
Content becomes structured execution.  
Execution becomes reviewable output.

---

## License

This project is released under the MIT License.

See `LICENSE` for details.

---

## Origin

Open Agentic CMO was originally built by Piggy Wallet as an internal autonomous content pipeline.

It is being open-sourced to help other builders explore deterministic, artifact-based, multi-agent marketing systems.

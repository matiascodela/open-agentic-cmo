# Open Agentic CMO

**Open Agentic CMO** is an open-source deterministic multi-agent CMO system for running a complete content marketing workflow:

```text
Research → Strategy → Content → Notion → Delivery
```

It was originally built for **Piggy Wallet**, a fintech + edtech product helping families in emerging markets protect savings from inflation and build healthier financial habits.

The system is designed to behave like an autonomous CMO team: it researches, synthesizes strategy, generates platform-native content, persists the output into Notion, and sends a delivery summary — all through structured agents, signals, artifacts, validation gates, and human-reviewable outputs.

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
- **Failure recovery** — invalid or incomplete states block safely instead of corrupting downstream execution

The goal is not just to generate content.

The goal is to build a reliable, auditable, autonomous marketing operating system.

---

## Core workflow

```text
Research → Strategy → Content → Notion → Delivery
```

### 1. Research

Babe analyzes available signals and produces a structured research artifact.

Output:

```text
audit
```

Signal:

```text
AUDIT_READY
```

---

### 2. Strategy

Porky synthesizes the research into a structured strategy artifact.

Output:

```text
strategy
```

Signal:

```text
SYNTHESIS_READY
```

---

### 3. Content

Hamm transforms the strategy into a structured content set.

Output:

```text
content_set
```

Signal:

```text
CONTENT_SET_READY
```

---

### 4. Notion

Pumba persists:

1. Final content into the Notion Content Pipeline
2. Babe’s research into a dedicated Notion page called `Research Babe`

Output:

```text
notion_content_pipeline
```

Signal:

```text
NOTION_SYNC_COMPLETE
```

---

### 5. Delivery

Pumba sends a Telegram summary and emits final delivery completion.

Output:

```text
telegram_delivery
```

Signal:

```text
DELIVERY_COMPLETE
```

---

## Quickstart

For a VPS-based setup using Paperclip as the agent control plane, start here:

```text
docs/10-paperclip-vps-quickstart.md
```

The quickstart covers:

- VPS preparation
- SSH access
- non-root user setup
- firewall setup
- Node.js 20+ and pnpm installation
- Paperclip installation
- secure SSH tunnel access
- Open Agentic CMO agent setup
- Notion credentials
- Telegram credentials
- controlled E2E test execution

For the first validation run, use:

```text
docs/examples/example-controlled-e2e-test.md
```

Recommended first test:

```text
3 days
1 X thread
1 X post
1 LinkedIn post
1 Research Babe page
1 Telegram summary
```

---

## Agents

Open Agentic CMO currently uses four core agents.

---

### Porky — Orchestrator / CEO-CMO

Porky owns orchestration, strategy synthesis, validation, reconciliation, and final workflow completion.

Responsibilities:

- Orchestration
- Signal scanning
- Artifact scanning
- State reconciliation
- Strategy synthesis
- Artifact validation
- Validation gates
- Delegation
- Idempotency
- Blocking invalid workflows
- Final summary

Porky does **not** create content, perform Babe’s research, persist to Notion, or send Telegram summaries.

Porky files:

```text
agents/porky/AGENTS.md
agents/porky/SOUL.md
agents/porky/TOOLS.md
agents/porky/HEARTBEAT.md
```

---

### Babe — Research / Audit Agent

Babe turns raw signals into structured research artifacts.

Responsibilities:

- Digital presence audit
- Research signal analysis
- Pattern detection
- Inconsistency detection
- Uncertainty and gap surfacing
- Opportunity identification
- Recommended content angles

Babe emits:

```text
AUDIT_READY
```

Babe file:

```text
agents/babe/AGENTS.md
```

---

### Hamm — Content Lead Agent

Hamm turns strategy into structured, publication-ready content.

Responsibilities:

- X posts
- X threads
- LinkedIn posts
- Content formatting
- Platform-native writing
- Content artifact generation

Hamm emits:

```text
CONTENT_SET_READY
```

Hamm file:

```text
agents/hamm/AGENTS.md
```

---

### Pumba — Notion Persistence / Delivery Agent

Pumba turns structured artifacts into persisted and delivered outputs.

Responsibilities:

- Persist content into Notion
- Persist Babe research into `Research Babe`
- Verify Notion structure
- Preserve thread ordering
- Avoid duplicates
- Send Telegram summary

Pumba emits:

```text
NOTION_SYNC_COMPLETE
DELIVERY_COMPLETE
```

Pumba file:

```text
agents/pumba/AGENTS.md
```

---

## System design principles

### Signal-driven

Every phase transition requires a valid signal.

Canonical signal format:

```text
SIGNAL:
type: <SIGNAL_TYPE>
producer: <AGENT_NAME>
issue: <PARENT_ISSUE_ID>
subtask_issue: <SUBTASK_ISSUE_ID>
status: <COMPLETE | FAILED>
artifact: <ARTIFACT_NAME>
timestamp: <TIMESTAMP>
```

Signals must be explicit, structured, and valid.

Inline or malformed signals should not be used for phase transitions.

---

### Artifact-driven

Signals alone are not enough.

Each completion signal must correspond to a valid structured artifact.

Example artifact header:

```text
ARTIFACT:
type: content_set
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
item_count: 3
content_items:
```

Artifacts are the source of truth for the workflow.

---

### Validation-first

The workflow never advances with missing, incomplete, or invalid artifacts.

If a phase fails validation, Porky blocks the workflow and requests a correction.

Blocking is preferred over corrupted downstream execution.

---

### Idempotent

The system is designed to avoid:

- Duplicate content generation
- Duplicate Notion pages
- Duplicate `Research Babe` pages
- Duplicate Telegram summaries
- Duplicate workflow signals
- Duplicate execution of completed valid phases

---

### Human-reviewable

The system does not only persist final content.

It also persists the research context behind the content so humans can understand the reasoning behind each campaign.

---

## Artifact types

The core artifact types are:

```text
audit
strategy
content_set
notion_content_pipeline
telegram_delivery
```

Each artifact type has a documented contract and a JSON schema.

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

Example header:

```text
ARTIFACT:
type: audit
issue: <PARENT_ISSUE_ID>
subtask_issue: <BABE_SUBTASK_ISSUE_ID>
entity: Piggy Wallet
timestamp:
sections:
```

Full example:

```text
docs/examples/example-audit-artifact.md
```

Schema:

```text
schemas/audit.schema.json
```

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

Example header:

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

Full example:

```text
docs/examples/example-strategy-artifact.md
```

Schema:

```text
schemas/strategy.schema.json
```

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

Example header:

```text
ARTIFACT:
type: content_set
issue: <PARENT_ISSUE_ID>
subtask_issue: <HAMM_SUBTASK_ISSUE_ID>
item_count: 3
content_items:
```

Full example:

```text
docs/examples/example-content-set-artifact.md
```

Schema:

```text
schemas/content-set.schema.json
```

---

## Notion content pipeline artifact

Produced or verified by Pumba.

This artifact verifies that:

- Content Pipeline entries exist
- Required Notion properties are populated
- Content is stored in page body
- Content is not stored in `Notes`
- Threads are split into ordered body blocks
- Research Babe page exists
- No duplicates exist

Schema:

```text
schemas/notion-content-pipeline.schema.json
```

---

## Telegram delivery artifact

Produced or verified by Pumba.

This artifact verifies that:

- Telegram summary was sent
- Delivery was not duplicated
- Summary references the parent issue
- Summary references Notion persistence
- Summary references Research Babe
- Summary includes item counts, platforms, and languages

Schema:

```text
schemas/telegram-delivery.schema.json
```

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

Signal examples:

```text
docs/examples/example-signals.md
```

Signal schema:

```text
schemas/signal.schema.json
```

---

## Notion persistence model

Pumba persists two types of Notion outputs.

---

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

---

### 2. Research Babe

For every workflow, Pumba creates or updates a page titled:

```text
Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>
```

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

```text
Date range: May 11–13, 2026
```

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

Full controlled E2E test:

```text
docs/examples/example-controlled-e2e-test.md
```

---

## Repository structure

```text
open-agentic-cmo/
├── agents/
│   ├── babe/
│   │   └── AGENTS.md
│   ├── hamm/
│   │   └── AGENTS.md
│   ├── porky/
│   │   ├── AGENTS.md
│   │   ├── HEARTBEAT.md
│   │   ├── SOUL.md
│   │   └── TOOLS.md
│   └── pumba/
│       └── AGENTS.md
├── docs/
│   ├── examples/
│   │   ├── example-audit-artifact.md
│   │   ├── example-content-set-artifact.md
│   │   ├── example-controlled-e2e-test.md
│   │   ├── example-signals.md
│   │   └── example-strategy-artifact.md
│   ├── 01-overview.md
│   ├── 02-architecture.md
│   ├── 03-agent-contracts.md
│   ├── 04-signal-contract.md
│   ├── 05-artifact-contracts.md
│   ├── 06-orchestration-loop.md
│   ├── 07-notion-persistence.md
│   ├── 08-testing-e2e.md
│   ├── 09-failure-recovery.md
│   └── 10-paperclip-vps-quickstart.md
├── schemas/
│   ├── audit.schema.json
│   ├── content-set.schema.json
│   ├── notion-content-pipeline.schema.json
│   ├── signal.schema.json
│   ├── strategy.schema.json
│   └── telegram-delivery.schema.json
├── .env.example
├── .gitignore
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── SECURITY.md
```

---

## Documentation

Core documentation:

```text
docs/01-overview.md
docs/02-architecture.md
docs/03-agent-contracts.md
docs/04-signal-contract.md
docs/05-artifact-contracts.md
docs/06-orchestration-loop.md
docs/07-notion-persistence.md
docs/08-testing-e2e.md
docs/09-failure-recovery.md
docs/10-paperclip-vps-quickstart.md
```

Examples:

```text
docs/examples/example-audit-artifact.md
docs/examples/example-strategy-artifact.md
docs/examples/example-content-set-artifact.md
docs/examples/example-signals.md
docs/examples/example-controlled-e2e-test.md
```

Schemas:

```text
schemas/signal.schema.json
schemas/audit.schema.json
schemas/strategy.schema.json
schemas/content-set.schema.json
schemas/notion-content-pipeline.schema.json
schemas/telegram-delivery.schema.json
```

Community and safety:

```text
SECURITY.md
CONTRIBUTING.md
CODE_OF_CONDUCT.md
```

---

## Environment variables

Do not commit real API keys, tokens, database IDs, or private URLs.

Use `.env.example` for placeholders:

```env
NOTION_API_KEY=your_notion_api_key_here
NOTION_CONTENT_DATABASE_ID=your_notion_content_database_id_here
NOTION_RESEARCH_DATABASE_ID=your_notion_research_database_id_here

TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
TELEGRAM_CHAT_ID=your_telegram_chat_id_here
```

Real credentials should live only in local or deployment environment files.

Never commit:

- `.env`
- API keys
- Notion tokens
- Telegram bot tokens
- database IDs
- private URLs
- private workflow logs

See:

```text
SECURITY.md
```

---

## Current status

The current system supports:

- Structured research artifacts
- Structured strategy artifacts
- Structured content artifacts
- Notion Content Pipeline verification artifacts
- Telegram delivery verification artifacts
- X posts
- X threads
- LinkedIn posts
- Notion Content Pipeline persistence
- Research Babe Notion page persistence
- Telegram delivery summaries
- Signal-based orchestration
- Validation gates
- Idempotency rules
- Failure recovery rules
- JSON schemas for all current core artifact types
- Controlled E2E testing documentation
- Paperclip VPS setup documentation

Piglet, the image generation agent, is currently excluded from the core workflow to preserve system stability.

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
- Builders designing deterministic multi-agent systems

---

## Contributing

Contributions should preserve the system’s contract-based design.

Before contributing, read:

```text
CONTRIBUTING.md
SECURITY.md
CODE_OF_CONDUCT.md
```

Good contributions improve:

- clarity
- determinism
- validation
- documentation
- examples
- schemas
- failure recovery
- human reviewability

Do not add real credentials, private data, or unsupported future claims to public documentation.

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

The workflow is only successful when the system can prove it is successful.

---

## License

This project is released under the MIT License.

See `LICENSE` for details.

---

## Origin

Open Agentic CMO was originally built by Piggy Wallet as an internal autonomous content pipeline.

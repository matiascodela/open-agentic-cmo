# Open Agentic CMO

**Open Agentic CMO** is an open-source deterministic multi-agent CMO system for running a complete content marketing workflow:

```text
Research → Strategy → Content → Notion → Delivery
```

It was originally built for **Piggy Wallet** — a fintech + edtech product helping families in emerging markets protect savings from inflation and build healthier financial habits.

Links:

- Website: https://www.piggywallet.app/
- Stellar app: https://stellar.piggywallet.app/

This repository currently provides the agent contracts, documentation, schemas, examples, and setup instructions for running the workflow in Paperclip.

It is not a standalone executable application.

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

## Install

Open Agentic CMO is designed to run in **Paperclip** using the agent contracts in this repository.

For the full VPS setup, see:

```text
docs/10-paperclip-vps-quickstart.md
```

### 1. Clone the repository

```bash
git clone https://github.com/matiascodela/open-agentic-cmo.git
cd open-agentic-cmo
```

### 2. Create your environment file

```bash
cp .env.example .env
```

### 3. Configure environment variables

Edit `.env` and add your own credentials:

```env
NOTION_API_KEY=your_notion_api_key_here
NOTION_CONTENT_DATABASE_ID=your_notion_content_database_id_here
NOTION_RESEARCH_DATABASE_ID=your_notion_research_database_id_here

TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
TELEGRAM_CHAT_ID=your_telegram_chat_id_here
```

Do not commit `.env`.

Real credentials should stay local or inside your deployment environment.

### 4. Install Paperclip

Follow the VPS quickstart:

```text
docs/10-paperclip-vps-quickstart.md
```

The guide covers:

- VPS preparation
- Node.js 20+ and pnpm installation
- Paperclip installation
- SSH tunnel access
- Paperclip workspace setup
- agent configuration
- Notion and Telegram setup

### 5. Add the agent instructions

Create the four core agents in Paperclip and paste the corresponding instruction files:

```text
agents/porky/AGENTS.md
agents/babe/AGENTS.md
agents/hamm/AGENTS.md
agents/pumba/AGENTS.md
```

Porky also includes supporting operating files:

```text
agents/porky/SOUL.md
agents/porky/TOOLS.md
agents/porky/HEARTBEAT.md
```

---

## Quickstart

After installing Paperclip and configuring the agents, run the controlled E2E test:

```text
docs/examples/example-controlled-e2e-test.md
```

Recommended first run:

```text
3 days
1 X thread
1 X post
1 LinkedIn post
1 Research Babe page
1 Telegram summary
```

The controlled E2E test validates:

- Babe produces a valid `audit` artifact
- Porky produces a valid `strategy` artifact
- Hamm produces a valid `content_set` artifact
- Pumba persists content into Notion
- Pumba creates or updates `Research Babe`
- Pumba sends a Telegram summary
- Porky validates every phase before completion

---

## Core workflow

```text
Research → Strategy → Content → Notion → Delivery
```

| Phase | Owner | Artifact | Signal |
|---|---|---|---|
| Research | Babe | `audit` | `AUDIT_READY` |
| Strategy | Porky | `strategy` | `SYNTHESIS_READY` |
| Content | Hamm | `content_set` | `CONTENT_SET_READY` |
| Notion | Pumba | `notion_content_pipeline` | `NOTION_SYNC_COMPLETE` |
| Delivery | Pumba | `telegram_delivery` | `DELIVERY_COMPLETE` |

---

## Agents

### Porky — Orchestrator / CEO-CMO

Porky owns orchestration, strategy synthesis, validation, reconciliation, delegation, and final workflow completion.

Porky does not create content, perform Babe’s research, persist to Notion, or send Telegram summaries.

Files:

```text
agents/porky/AGENTS.md
agents/porky/SOUL.md
agents/porky/TOOLS.md
agents/porky/HEARTBEAT.md
```

### Babe — Research / Audit Agent

Babe turns raw signals into structured research artifacts.

File:

```text
agents/babe/AGENTS.md
```

### Hamm — Content Lead Agent

Hamm turns strategy into structured, publication-ready content.

File:

```text
agents/hamm/AGENTS.md
```

### Pumba — Notion Persistence / Delivery Agent

Pumba turns structured artifacts into persisted and delivered outputs.

File:

```text
agents/pumba/AGENTS.md
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
docs/examples/example-controlled-e2e-test.md
docs/examples/example-audit-artifact.md
docs/examples/example-strategy-artifact.md
docs/examples/example-content-set-artifact.md
docs/examples/example-signals.md
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

## Security

Do not commit real API keys, tokens, database IDs, private URLs, private workflow logs, or production screenshots.

Use `.env.example` for placeholders only.

Read:

```text
SECURITY.md
```

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

Do not add unsupported future claims to core documentation.

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

## Philosophy

This project is based on a simple idea:

> Agents should not just write.  
> Agents should produce structured, validated, traceable work.

Open Agentic CMO treats marketing as an operating system:

```text
Research becomes strategy.
Strategy becomes content.
Content becomes structured execution.
Execution becomes reviewable output.
```

The workflow is only successful when the system can prove it is successful.

---

## License

This project is released under the MIT License.

See `LICENSE` for details.

---

## Origin

Open Agentic CMO was originally built by Piggy Wallet as an internal autonomous content pipeline.

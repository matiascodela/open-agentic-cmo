# Overview

Open Agentic CMO is an open-source deterministic multi-agent system for running a complete content marketing workflow.

The core workflow is:

Research → Strategy → Content → Notion → Delivery

The system was originally built for Piggy Wallet, a fintech + edtech product helping families in emerging markets protect savings from inflation and build healthier financial habits.

Open Agentic CMO is designed to behave like an autonomous CMO team. It researches, synthesizes strategy, generates platform-native content, persists outputs into Notion, and sends delivery summaries through structured agents, signals, artifacts, and validation gates.

---

## What problem does it solve?

Many AI agent systems fail because they rely on loose text outputs, implicit assumptions, and vague completion criteria.

Common failure modes include:

- Agents saying work is complete without producing usable outputs
- Signals being emitted before artifacts exist
- Content being generated but not persisted correctly
- Duplicate Notion entries
- Missing fields such as `Version`, `Week`, or `Scheduled Date`
- Content being stored in the wrong place, such as `Notes` instead of the Notion page body
- Research context being lost after content generation
- Downstream agents advancing based on summaries instead of validated artifacts

Open Agentic CMO solves these problems by making every workflow transition explicit, structured, and verifiable.

---

## Core idea

The system is built around a simple principle:

> Agents should not just write.  
> Agents should produce structured, validated, traceable work.

Instead of relying on free-form outputs, each agent produces a formal artifact.

Instead of relying on “looks good” reasoning, each workflow phase requires a valid signal and a validated artifact before the next phase can begin.

---

## Core workflow

The workflow has five main phases.

---

### 1. Research

Research is handled by Babe.

Babe analyzes available signals across sources such as:

- X / Twitter
- Website
- Product surfaces
- Brand context
- Market signals
- Competitive references

Babe produces a structured `audit` artifact.

The audit artifact includes:

- Summary
- Key signals
- Inconsistencies
- Uncertainty / data gaps
- Opportunities
- Recommended content angles

Babe emits:

`AUDIT_READY`

---

### 2. Strategy

Strategy is handled by Porky.

Porky takes Babe’s audit artifact and synthesizes it into a structured `strategy` artifact.

The strategy artifact includes:

- Date range
- Expected content items
- Messaging pillars
- Content angles
- Weekly structure
- Brand constraints
- Content rules

Porky emits:

`SYNTHESIS_READY`

---

### 3. Content

Content is handled by Hamm.

Hamm takes the strategy artifact and generates a structured `content_set` artifact.

The content set may include:

- X posts
- X threads
- LinkedIn posts

Each content item includes required metadata such as:

- Title
- Platform
- Language
- Content type
- Scheduled date
- Vertical
- Version
- Week
- Tags and mentions

For posts, Hamm provides a complete body.

For threads, Hamm provides an ordered list of tweets.

Hamm emits:

`CONTENT_SET_READY`

---

### 4. Notion

Notion persistence is handled by Pumba.

Pumba persists two types of outputs.

First, it persists Hamm’s content into the Notion Content Pipeline.

Each content item becomes a Notion page with structured properties and body content.

Second, it persists Babe’s research into a dedicated page called:

`Research Babe`

This ensures humans can review not only the final content, but also the research and analysis that informed the strategy.

Pumba emits:

`NOTION_SYNC_COMPLETE`

---

### 5. Delivery

Delivery is also handled by Pumba.

Pumba sends a Telegram summary after Notion persistence has been verified.

The summary includes:

- Parent issue id
- Date range
- Number of content items
- Number of posts
- Number of threads
- Platforms used
- Languages used
- Notion Content Pipeline status
- Research Babe page status
- Duplicate check result

Pumba emits:

`DELIVERY_COMPLETE`

---

## Agents

Open Agentic CMO currently includes four core agents.

---

## Porky — Orchestrator / CEO-CMO

Porky is the orchestrator and validation gatekeeper.

Porky is responsible for:

- Running the orchestration loop
- Scanning parent issues and subtasks
- Validating signals
- Validating artifacts
- Synthesizing strategy
- Delegating work to other agents
- Reconciling state
- Blocking invalid transitions
- Preventing duplicate or corrupted execution

Porky does not perform research, write final content, persist to Notion, or send Telegram summaries.

---

## Babe — Research / Audit Agent

Babe is the research layer.

Babe is responsible for:

- Running structured research
- Auditing digital presence
- Identifying key signals
- Surfacing uncertainty
- Detecting inconsistencies
- Producing an `audit` artifact

Babe does not create content, define strategy, orchestrate workflows, persist to Notion, or send Telegram summaries.

---

## Hamm — Content Lead Agent

Hamm is the content generation layer.

Hamm is responsible for:

- Turning strategy into content
- Generating X posts
- Generating X threads
- Generating LinkedIn posts
- Producing a structured `content_set` artifact

Hamm does not perform research, define strategy, orchestrate workflows, persist to Notion, or send Telegram summaries.

---

## Pumba — Notion Persistence / Delivery Agent

Pumba is the final execution layer.

Pumba is responsible for:

- Validating content and research artifacts
- Persisting content into Notion
- Persisting Babe’s research into `Research Babe`
- Preserving thread structure
- Preventing duplicates
- Sending Telegram summaries
- Emitting Notion and delivery completion signals

Pumba does not create content, define strategy, perform research, or orchestrate workflows.

---

## Signals

Signals are explicit workflow transition markers.

A signal tells the system that a phase has reached a defined state.

Example signal:

SIGNAL:  
type: CONTENT_SET_READY  
producer: Hamm  
issue: `<PARENT_ISSUE_ID>`  
subtask_issue: `<SUBTASK_ISSUE_ID>`  
status: COMPLETE  
artifact: content_set  
timestamp: `<TIMESTAMP>`

Signals are required, but signals alone are not enough.

A valid signal must correspond to a valid artifact.

---

## Artifacts

Artifacts are structured outputs produced by agents.

Core artifact types:

- `audit`
- `strategy`
- `content_set`
- `notion_content_pipeline`
- `telegram_delivery`

Artifacts are the source of truth for the workflow.

If a signal exists but the artifact is missing, the workflow must not advance.

If an artifact exists but is incomplete or invalid, the workflow must block.

---

## Validation gates

Validation gates are hard checks between workflow phases.

Before advancing, Porky validates that the required artifact exists and meets the expected contract.

Examples:

- `AUDIT_READY` is valid only if an `audit` artifact exists.
- `SYNTHESIS_READY` is valid only if a `strategy` artifact exists.
- `CONTENT_SET_READY` is valid only if a `content_set` artifact exists.
- `NOTION_SYNC_COMPLETE` is valid only if Notion content and Research Babe are correctly persisted.
- `DELIVERY_COMPLETE` is valid only if the Telegram summary was sent correctly.

This prevents the system from advancing with incomplete or corrupted data.

---

## Notion persistence model

Pumba persists content into the Notion Content Pipeline.

Each content item becomes one Notion page.

Required properties include:

- Title
- Platform
- Status
- Scheduled Date
- Vertical
- Version
- Week
- Content Type

For threads, the page may also include:

- Thread ID
- Thread Order

The content body must be written into the Notion page body.

Content must not be stored in `Notes`.

---

## Research Babe

Research Babe is a dedicated Notion page created or updated for each workflow.

Its purpose is to preserve Babe’s research and make it human-reviewable.

Research Babe includes:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Source / Workflow Metadata

This makes the system more transparent and reviewable.

Humans can approve content while also understanding the research and context that shaped it.

---

## Idempotency

Open Agentic CMO is designed to avoid duplicate work.

The system should not:

- Generate the same content twice
- Create duplicate Notion pages
- Create duplicate Research Babe pages
- Send duplicate Telegram summaries
- Emit duplicate completion signals
- Re-run completed valid phases

If a valid artifact already exists, the system should reuse it.

If a page already exists in Notion, Pumba should update it instead of creating a duplicate.

---

## Failure handling

When something is missing or invalid, the system blocks instead of advancing.

Examples of blocking conditions:

- Missing parent issue id
- Missing subtask issue id
- Missing artifact
- Invalid signal
- Incomplete content item
- Missing scheduled date
- Missing Version or Week
- Content stored in Notes
- Missing Research Babe page
- Duplicate Notion entries
- Failed Telegram delivery

A blocked workflow is preferable to a corrupted workflow.

---

## Current scope

The current core system supports:

- Research artifact generation
- Strategy artifact generation
- Content set artifact generation
- X posts
- X threads
- LinkedIn posts
- Notion Content Pipeline persistence
- Research Babe persistence
- Telegram delivery summaries
- Signal-based orchestration
- Artifact validation
- Idempotency and reconciliation

Image generation is currently excluded from the core workflow to preserve stability.

---

## Design philosophy

Open Agentic CMO treats marketing as an operating system.

Research becomes strategy.  
Strategy becomes content.  
Content becomes structured execution.  
Execution becomes reviewable output.

The system is intentionally strict because reliable agentic workflows require more than good prompts.

They require contracts, validation, and deterministic execution.

# Architecture

Open Agentic CMO is a deterministic multi-agent content operations system.

Its architecture is built around four core ideas:

1. Agents have strict responsibilities.
2. Workflow transitions are driven by signals.
3. Work products are stored as structured artifacts.
4. Porky validates every phase before the workflow can advance.

The system is intentionally strict because autonomous workflows fail when agents advance based on summaries, assumptions, or incomplete outputs.

---

## High-level architecture

The core workflow is:

**Research → Strategy → Content → Notion → Delivery**

Each phase is owned by a specific agent.

| Phase | Owner | Output |
|---|---|---|
| Research | Babe | `audit` artifact |
| Strategy | Porky | `strategy` artifact |
| Content | Hamm | `content_set` artifact |
| Notion | Pumba | Notion Content Pipeline + Research Babe |
| Delivery | Pumba | Telegram summary |

---

## System diagram

```text
Parent Issue
    ↓
Porky — Orchestrator / Validator
    ↓
Babe → audit artifact → AUDIT_READY
    ↓
Porky → strategy artifact → SYNTHESIS_READY
    ↓
Hamm → content_set artifact → CONTENT_SET_READY
    ↓
Pumba → Notion Content Pipeline + Research Babe → NOTION_SYNC_COMPLETE
    ↓
Pumba → Telegram summary → DELIVERY_COMPLETE
    ↓
Porky → Final summary + workflow completion
```

---

## Agent responsibilities

### Porky — Orchestrator / CEO-CMO

Porky is the central orchestrator and validation gatekeeper.

Porky owns:

- Workflow state
- Phase transitions
- Signal scanning
- Artifact validation
- Strategy synthesis
- Delegation
- Reconciliation
- Blocking invalid execution
- Final completion checks

Porky does not own:

- Research execution
- Content generation
- Notion persistence
- Telegram delivery

Porky is the only agent that can decide whether the workflow is allowed to advance.

---

### Babe — Research / Audit Agent

Babe owns the research phase.

Babe produces:

- `audit` artifact
- `AUDIT_READY` signal

Babe analyzes:

- X / Twitter
- Website
- Product surfaces
- Market signals
- Competitive signals
- Audience patterns
- Uncertainty and data gaps

Babe does not create content or define strategy.

---

### Hamm — Content Lead Agent

Hamm owns the content generation phase.

Hamm consumes:

- `strategy` artifact

Hamm produces:

- `content_set` artifact
- `CONTENT_SET_READY` signal

The content set may include:

- X posts
- X threads
- LinkedIn posts

Hamm does not persist to Notion or send delivery summaries.

---

### Pumba — Notion Persistence / Delivery Agent

Pumba owns persistence and delivery.

Pumba consumes:

- `audit` artifact
- `content_set` artifact

Pumba produces or verifies:

- Notion Content Pipeline entries
- Research Babe page
- Telegram summary
- `NOTION_SYNC_COMPLETE` signal
- `DELIVERY_COMPLETE` signal

Pumba does not create content, define strategy, or orchestrate workflows.

---

## Phase architecture

### Phase 1 — Research

**Owner:** Babe

**Input:**

- Task from Porky
- Target entity
- Optional scope
- Parent issue ID
- Subtask issue ID

**Output:**

- `audit` artifact
- `AUDIT_READY` signal

The audit artifact must include:

- Summary
- Key signals
- Inconsistencies
- Uncertainty
- Opportunities
- Recommended content angles

Porky validates the audit before advancing.

---

### Phase 2 — Strategy

**Owner:** Porky

**Input:**

- Valid `audit` artifact

**Output:**

- `strategy` artifact
- `SYNTHESIS_READY` signal

The strategy artifact must include:

- Date range
- Expected items
- Messaging pillars
- Content angles
- Weekly structure
- Brand constraints
- Content rules

Porky validates its own strategy artifact before delegating content generation.

---

### Phase 3 — Content

**Owner:** Hamm

**Input:**

- Valid `strategy` artifact
- Date range
- Content rules
- Platform mix
- Parent issue ID
- Subtask issue ID

**Output:**

- `content_set` artifact
- `CONTENT_SET_READY` signal

Each content item must include:

- Title
- Platform
- Language
- Content type
- Scheduled date
- Vertical
- Version
- Week
- Tags and mentions

Posts must include:

- Body

Threads must include:

- Thread ID
- Ordered tweets

Porky validates the content set before delegating to Pumba.

---

### Phase 4 — Notion

**Owner:** Pumba

**Input:**

- Valid `content_set` artifact
- Valid `audit` artifact

**Output:**

- Content Pipeline pages in Notion
- Research Babe page in Notion
- `NOTION_SYNC_COMPLETE` signal

Pumba must persist both:

1. Content items into the Notion Content Pipeline
2. Babe research into the Research Babe page

Porky validates both before allowing delivery completion.

---

### Phase 5 — Delivery

**Owner:** Pumba

**Input:**

- Verified Notion persistence

**Output:**

- Telegram summary
- `DELIVERY_COMPLETE` signal

The Telegram summary should include:

- Parent issue ID
- Date range
- Total content items
- Number of posts
- Number of threads
- Platforms used
- Languages used
- Notion status
- Research Babe status
- Duplicate check result

Porky validates final delivery before completing the workflow.

---

## Signal architecture

Signals are explicit workflow transition markers.

A signal does not contain the full work product.

A signal only declares that a specific artifact is ready, blocked, delivered, or failed.

Example:

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
issue: <PARENT_ISSUE_ID>
subtask_issue: <SUBTASK_ISSUE_ID>
status: COMPLETE
artifact: content_set
timestamp: <TIMESTAMP>
```

A valid signal must include:

- Type
- Producer
- Issue
- Status
- Artifact
- Timestamp

Preferred:

- Subtask issue

Signals are extracted from:

- Parent issue comments
- Subtask comments

Porky normalizes all valid signals to the parent workflow.

---

## Artifact architecture

Artifacts are structured work products.

They are the source of truth for the workflow.

Core artifacts:

| Artifact | Producer | Consumer |
|---|---|---|
| `audit` | Babe | Porky, Pumba |
| `strategy` | Porky | Hamm |
| `content_set` | Hamm | Porky, Pumba |
| `notion_content_pipeline` | Pumba | Porky |
| `telegram_delivery` | Pumba | Porky |

A signal without an artifact is incomplete.

An artifact without a signal can be reconciled if it passes validation.

---

## Validation architecture

Validation is centralized in Porky.

Before each phase transition, Porky checks:

1. Does the required signal exist?
2. Does the required artifact exist?
3. Does the artifact pass the contract?
4. Has the phase already completed?
5. Would executing the next phase duplicate work?
6. Is downstream state safe?

If validation fails, the workflow blocks.

Blocking is intentional.

A blocked workflow is safer than corrupted output.

---

## Reconciliation architecture

The system supports reconciliation.

This means Porky can recover from partial execution if valid artifacts exist.

### Signal missing, artifact exists

If Hamm produced a valid `content_set` artifact but forgot to emit `CONTENT_SET_READY`, Porky can:

1. Validate the artifact
2. Register internal completion
3. Request signal propagation
4. Continue safely

### Signal exists, artifact missing

If Hamm emits `CONTENT_SET_READY` but no `content_set` artifact exists, Porky must:

1. Treat the signal as incomplete
2. Block the workflow
3. Request the full artifact
4. Prevent Pumba from running

### Artifact invalid

If the artifact exists but violates the contract, Porky blocks the workflow.

---

## Idempotency architecture

Open Agentic CMO is designed to avoid duplicate work.

Each phase checks for existing valid outputs before executing.

Examples:

- Babe should not rerun research if a valid audit already exists.
- Hamm should not regenerate content if a valid content set already exists.
- Pumba should update existing Notion entries instead of creating duplicates.
- Pumba should not resend Telegram summaries if a valid summary was already sent.
- Porky should not re-trigger completed phases.

Deduplication keys include:

For content:

```text
parent_issue_id + scheduled_date + platform + title
```

For Research Babe:

```text
parent_issue_id + babe_research
```

---

## Notion architecture

Notion is used as the human-reviewable workspace.

Pumba writes to two areas.

### Content Pipeline

Each content item becomes a Notion page.

Required properties:

- Title
- Platform
- Status
- Scheduled Date
- Vertical
- Version
- Week
- Content Type

For threads:

- Thread ID
- Thread Order, if applicable

Body rules:

- Posts are written into the page body
- Threads are written as ordered body blocks
- Content is not stored in Notes

---

### Research Babe

Each workflow also gets a research page titled:

```text
Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>
```

This page contains:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Source / Workflow Metadata

This makes the workflow auditable by humans.

---

## Blocking architecture

The system blocks when required inputs or outputs are missing.

Examples:

- Missing parent issue ID
- Missing subtask issue ID
- Malformed signal
- Missing artifact
- Incomplete audit
- Missing strategy
- Content set without full items
- Missing scheduled dates
- Missing Version or Week
- Notion content stored in Notes
- Missing Research Babe page
- Duplicate content entries
- Duplicate Telegram delivery

A block must include:

- Blocked phase
- Responsible agent
- Missing or invalid requirement
- Required next action

---

## Final completion architecture

The workflow is complete only when every phase has passed validation.

Completion requires:

- Valid audit artifact
- Valid strategy artifact
- Valid content set artifact
- Verified Notion Content Pipeline entries
- Verified Research Babe page
- Verified Telegram delivery
- All required signals
- No duplicate content entries
- No duplicate Research Babe page
- No corrupted data

Porky posts a final summary only after all gates pass.

---

## Current scope

The current architecture intentionally excludes image generation.

Piglet, the image generation agent, was removed from the core workflow to protect system stability.

Future versions may reintroduce image generation as a separate module with:

- Fallback providers
- Hard output validation
- No blocking dependency on the core content pipeline

---

## Design tradeoffs

Open Agentic CMO favors reliability over flexibility.

This means:

- Stricter contracts
- More validation
- More blocking
- Less free-form behavior
- Less agent autonomy inside each phase

This is intentional.

The system is designed for repeatable content operations, not one-off creative improvisation.

---

## Architectural principle

The core architectural principle is:

> Agents should not advance workflows by saying work is done.  
> Agents should advance workflows by producing valid artifacts that pass validation.

This makes the system deterministic, auditable, and safer to operate.

# Agent Contracts

Open Agentic CMO is built around strict agent contracts.

Each agent has a clearly defined role, input contract, output contract, allowed signals, forbidden actions, and completion criteria.

The goal is to prevent ambiguity.

Agents should not “help wherever needed.”  
Agents should perform only their assigned responsibility and produce structured, validated outputs.

---

## Why agent contracts matter

Multi-agent systems often fail because agents blur responsibilities.

Common failure modes include:

- Research agents writing content
- Content agents inventing strategy
- Persistence agents modifying copy
- Orchestrators performing specialist work
- Agents claiming completion without producing artifacts
- Downstream agents running before upstream outputs are valid

Open Agentic CMO avoids this by enforcing strict contracts.

Each agent must:

1. Operate only within its role.
2. Consume only valid inputs.
3. Produce a structured artifact.
4. Emit only allowed signals.
5. Propagate signals correctly.
6. Stop when its responsibility ends.

---

## Contract model

Every agent contract defines:

- Role
- Responsibilities
- Forbidden actions
- Required inputs
- Required outputs
- Artifact contract
- Signal contract
- Completion rule
- Reconciliation behavior

The current core agents are:

| Agent | Role | Primary Artifact | Main Signal |
|---|---|---|---|
| Porky | Orchestrator / CEO-CMO | `strategy` | `SYNTHESIS_READY` |
| Babe | Research / Audit | `audit` | `AUDIT_READY` |
| Hamm | Content Lead | `content_set` | `CONTENT_SET_READY` |
| Pumba | Persistence / Delivery | `notion_content_pipeline`, `telegram_delivery` | `NOTION_SYNC_COMPLETE`, `DELIVERY_COMPLETE` |

---

## Porky contract

### Role

Porky is the orchestrator, strategy synthesizer, and validation gatekeeper.

Porky owns the full workflow:

**Research → Strategy → Content → Notion → Delivery**

Porky does not own the specialist execution of every phase. Instead, Porky validates state and delegates work to the correct agent.

---

### Responsibilities

Porky is responsible for:

- Running the orchestration loop
- Scanning parent issue and subtask comments
- Extracting signals
- Extracting artifacts
- Normalizing workflow state
- Validating artifacts
- Synthesizing strategy from Babe’s audit
- Delegating research to Babe
- Delegating content generation to Hamm
- Delegating Notion persistence and delivery to Pumba
- Blocking invalid transitions
- Preventing duplicate execution
- Posting the final workflow summary

---

### Forbidden actions

Porky must not:

- Perform Babe’s research work
- Produce Hamm’s final content set
- Persist content to Notion
- Send Telegram summaries
- Advance based on task status alone
- Advance based on summaries alone
- Advance based on agent claims alone
- Ignore missing artifacts

---

### Inputs

Porky consumes:

- Parent issue
- Subtask comments
- Signals
- Artifacts
- Babe audit artifact
- Hamm content set artifact
- Pumba Notion and delivery outputs

---

### Outputs

Porky produces:

- Strategy artifact
- `SYNTHESIS_READY` signal
- Blocking messages when validation fails
- Final workflow summary

---

### Primary artifact

Porky produces the `strategy` artifact.

Required fields:

- `date_range`
- `expected_items`
- `messaging_pillars`
- `content_angles`
- `weekly_structure`
- `brand_constraints`
- `content_rules`

---

### Allowed signals

Porky may emit:

- `SYNTHESIS_READY`
- `BLOCKED`
- `TIMEOUT_DETECTED`
- `RECOVERY_IN_PROGRESS`
- `RECOVERY_COMPLETE`

---

### Completion rule

Porky’s workflow is complete only when:

- A valid audit artifact exists
- A valid strategy artifact exists
- A valid content set artifact exists
- Notion persistence is verified
- Research Babe page exists
- Telegram delivery is verified
- All required signals exist
- No duplicate or corrupted outputs exist

---

## Babe contract

### Role

Babe is the research and audit agent.

Babe transforms raw signals into a structured research artifact.

---

### Responsibilities

Babe is responsible for:

- Running structured research
- Auditing digital presence signals
- Analyzing X / Twitter
- Analyzing website and product surfaces
- Detecting patterns and inconsistencies
- Identifying opportunities
- Surfacing uncertainty and data gaps
- Producing the audit artifact
- Emitting `AUDIT_READY`

---

### Forbidden actions

Babe must not:

- Create content drafts
- Define strategy
- Decide campaign priorities
- Persist anything to Notion
- Send Telegram summaries
- Orchestrate workflows
- Generate images

---

### Inputs

Babe consumes:

- Task from Porky
- Target entity
- Optional scope
- Parent issue ID
- Subtask issue ID

---

### Outputs

Babe produces:

- `audit` artifact
- `AUDIT_READY` signal

---

### Primary artifact

Babe produces the `audit` artifact.

Required sections:

- `summary`
- `key_signals`
- `inconsistencies`
- `uncertainty`
- `opportunities`
- `recommended_content_angles`

---

### Allowed signals

Babe may emit:

- `AUDIT_READY`
- `BLOCKED`

Babe must not emit:

- `SYNTHESIS_READY`
- `CONTENT_SET_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

---

### Completion rule

Babe is complete only when:

- The full audit artifact is published
- All required sections exist
- `AUDIT_READY` is emitted
- The same signal is propagated to the parent issue

---

## Hamm contract

### Role

Hamm is the content generation agent.

Hamm transforms Porky’s strategy artifact into structured, platform-native content.

---

### Responsibilities

Hamm is responsible for:

- Generating X posts
- Generating X threads
- Generating LinkedIn posts
- Following the strategy artifact
- Producing publication-ready content
- Producing structured content metadata
- Producing the `content_set` artifact
- Emitting `CONTENT_SET_READY`

---

### Forbidden actions

Hamm must not:

- Perform research
- Define strategy
- Persist content to Notion
- Send Telegram summaries
- Orchestrate workflows
- Generate images
- Invent unsupported product claims

---

### Inputs

Hamm consumes:

- Strategy artifact
- Optional CEO brief
- Date range
- Content rules
- Platform mix
- Parent issue ID
- Subtask issue ID

---

### Outputs

Hamm produces:

- `content_set` artifact
- `CONTENT_SET_READY` signal

---

### Primary artifact

Hamm produces the `content_set` artifact.

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

---

### Content format requirements

Content types:

- `post`
- `thread`

Threads must include:

- 4–7 tweets
- Ordered tweet array
- Clear narrative progression
- No duplicate tweets

Thread structure should generally follow:

1. Hook
2. Problem
3. Insight
4. Solution
5. CTA

---

### Allowed signals

Hamm may emit:

- `CONTENT_SET_READY`
- `BLOCKED`

Hamm must not emit:

- `AUDIT_READY`
- `SYNTHESIS_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

---

### Completion rule

Hamm is complete only when:

- The full `content_set` artifact is published
- All content items are present
- `item_count` matches actual items
- Schema is valid
- No duplicates exist
- `CONTENT_SET_READY` is emitted
- The same signal is propagated to the parent issue

---

## Pumba contract

### Role

Pumba is the Notion persistence and delivery agent.

Pumba turns validated artifacts into persisted, reviewable, distribution-ready outputs.

---

### Responsibilities

Pumba is responsible for:

- Validating the `content_set` artifact
- Validating the `audit` artifact
- Persisting content into the Notion Content Pipeline
- Persisting Babe research into the Research Babe page
- Ensuring content is stored in Notion page bodies
- Ensuring content is not stored in Notes
- Preserving thread order as body blocks
- Preventing duplicate content entries
- Preventing duplicate Research Babe pages
- Sending Telegram summaries
- Emitting `NOTION_SYNC_COMPLETE`
- Emitting `DELIVERY_COMPLETE`

---

### Forbidden actions

Pumba must not:

- Create content
- Rewrite content creatively
- Define strategy
- Perform research
- Orchestrate workflows
- Invent missing fields
- Persist incomplete artifacts
- Mark completion before Notion verification

---

### Inputs

Pumba consumes:

- `CONTENT_SET_READY` signal
- `content_set` artifact
- `AUDIT_READY` signal
- `audit` artifact
- Optional strategy artifact
- Parent issue ID
- Subtask issue ID

---

### Outputs

Pumba produces or verifies:

- Notion Content Pipeline entries
- Research Babe page
- Telegram summary
- `NOTION_SYNC_COMPLETE` signal
- `DELIVERY_COMPLETE` signal

---

### Notion Content Pipeline contract

Each content item must become one Notion page.

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
- `Thread Order`, if applicable

Content must be written into the page body.

Content must not be written into `Notes`.

---

### Research Babe contract

Pumba must create or update a Notion page titled:

`Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>`

If date range is unavailable:

`Research Babe — <PARENT_ISSUE_ID>`

The page must include:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Source / Workflow Metadata

---

### Allowed signals

Pumba may emit:

- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`
- `BLOCKED`

Pumba must not emit:

- `AUDIT_READY`
- `SYNTHESIS_READY`
- `CONTENT_SET_READY`

---

### Completion rule

Pumba is complete only when:

- All content is correctly persisted in Notion
- Research Babe page exists
- No duplicates exist
- Content is in page body, not Notes
- Threads are split into ordered body blocks
- Telegram summary is sent once
- `NOTION_SYNC_COMPLETE` is emitted and propagated
- `DELIVERY_COMPLETE` is emitted and propagated

---

## Cross-agent rules

### Artifact before signal

Agents must publish the full artifact before emitting completion signals.

Invalid:

- “Done”
- “Content is available in comments”
- “Research completed”
- Signal without artifact

Valid:

1. Full artifact is published.
2. Signal is emitted.
3. Same signal is propagated to parent issue.

---

### No role leakage

Agents must not perform work owned by other agents.

Examples:

- Babe must not write final content.
- Hamm must not perform research.
- Pumba must not invent missing content.
- Porky must not skip validation.

---

### Parent issue propagation

After emitting a signal in a subtask, the agent must post the exact same signal in the parent issue.

The signal must not be modified.

---

### Blocking over corruption

If required input is missing or invalid, the agent must block.

Blocking is preferred over:

- inventing missing data
- creating partial artifacts
- persisting incomplete content
- advancing with uncertainty
- silently skipping requirements

---

## Contract enforcement

Porky enforces contracts.

Before advancing, Porky validates:

- signal format
- artifact existence
- artifact completeness
- artifact schema
- phase ownership
- Notion state
- delivery state

If any contract fails, Porky blocks the workflow and identifies:

- blocked phase
- responsible agent
- missing or invalid requirement
- required next action

---

## Summary

Open Agentic CMO works because every agent has a narrow role and a strict contract.

The system is not designed around agent creativity alone.

It is designed around:

- structured inputs
- structured outputs
- explicit signals
- validation gates
- deterministic orchestration
- human-reviewable execution

This contract-based design is what makes the system reliable enough for repeatable content operations.

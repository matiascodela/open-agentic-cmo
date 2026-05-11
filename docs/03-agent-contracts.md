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
- Agents posting final outputs only on subtasks
- Orchestrators creating title-only subtasks with missing context
- Persistence agents running before content exists
- Research agents spending the run on environment discovery instead of producing the audit

Open Agentic CMO avoids this by enforcing strict contracts.

Each agent must:

1. Operate only within its role.
2. Consume only valid inputs.
3. Produce a structured artifact.
4. Emit only allowed signals.
5. Post artifacts and signals on the parent issue.
6. Stop when its responsibility ends.

---

## Canonical workflow order

The workflow must run sequentially:

```text
Porky → Babe → Porky → Hamm → Porky → Pumba → Porky
```

Detailed order:

1. Porky starts the parent workflow.
2. Porky delegates research to Babe only.
3. Babe completes research.
4. Babe posts the full `audit` artifact and `AUDIT_READY` signal on the original parent issue.
5. Porky validates Babe’s audit artifact.
6. Porky synthesizes the `strategy` artifact.
7. Porky emits `SYNTHESIS_READY` on the original parent issue.
8. Porky delegates content generation to Hamm only after strategy validation passes.
9. Hamm completes content.
10. Hamm posts the full `content_set` artifact and `CONTENT_SET_READY` signal on the original parent issue.
11. Porky validates Hamm’s content set artifact.
12. Porky delegates Notion persistence and delivery to Pumba only after content validation passes.
13. Pumba persists Babe’s research and Hamm’s content to Notion.
14. Pumba posts `NOTION_SYNC_COMPLETE` and `DELIVERY_COMPLETE` on the original parent issue.
15. Porky validates Notion persistence and Telegram delivery.
16. Porky posts final summary and completes the workflow.

No downstream agent should be assigned before its required upstream artifact exists and has been validated.

---

## Parent issue as source of truth

The original parent issue is the canonical workflow record.

Subtasks may exist for specialist execution, but final workflow state must be visible on the parent issue.

Required parent issue evidence:

### Babe completion

```text
ARTIFACT:
type: audit
...
```

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
...
```

### Porky strategy completion

```text
ARTIFACT:
type: strategy
...
```

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
...
```

### Hamm completion

```text
ARTIFACT:
type: content_set
...
```

```text
SIGNAL:
type: CONTENT_SET_READY
producer: Hamm
...
```

### Pumba completion

```text
ARTIFACT:
type: notion_content_pipeline
...
```

```text
SIGNAL:
type: NOTION_SYNC_COMPLETE
producer: Pumba
...
```

```text
ARTIFACT:
type: telegram_delivery
...
```

```text
SIGNAL:
type: DELIVERY_COMPLETE
producer: Pumba
...
```

If an artifact or signal exists only on a subtask, Porky may use it for reconciliation, but final completion requires parent issue visibility.

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

```text
Research → Strategy → Content → Notion → Delivery
```

Porky does not own the specialist execution of every phase.

Instead, Porky:

- validates workflow state
- synthesizes strategy
- delegates the next valid phase
- blocks unsafe transitions
- ensures the workflow completes correctly

---

### Responsibilities

Porky is responsible for:

- Running the orchestration loop
- Scanning parent issue and subtask comments
- Extracting signals
- Extracting artifacts
- Normalizing workflow state
- Validating artifacts
- Validating signal format
- Synthesizing strategy from Babe’s audit
- Applying the Critical Fact / Brand Risk Gate
- Delegating research to Babe
- Delegating content generation to Hamm
- Delegating Notion persistence and delivery to Pumba
- Creating complete phase-specific subtask descriptions
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
- Create all downstream subtasks at workflow initialization
- Create title-only subtasks
- Advance based on task status alone
- Advance based on summaries alone
- Advance based on agent claims alone
- Ignore missing artifacts
- Ignore unresolved publishing risks

---

### Sequential delegation rule

Porky must delegate work sequentially.

At workflow start:

- create or activate only Babe
- do not create Hamm
- do not create Pumba

After Babe completes:

- validate the audit artifact
- synthesize strategy
- emit `SYNTHESIS_READY`
- create Hamm only after strategy validation passes

After Hamm completes:

- validate the `content_set` artifact
- create Pumba only after content validation passes

After Pumba completes:

- validate Notion persistence
- validate Research Babe
- validate Telegram delivery
- complete the workflow

---

### Subtask description rule

Porky must never create a title-only subtask.

Every delegated subtask must include enough context for the assigned agent to execute deterministically.

A valid subtask description must include:

- `parent_issue_id`
- phase name
- assigned agent
- objective
- workflow context
- source parent brief
- input artifacts required
- output artifact required
- expected signal
- parent task source-of-truth rule
- validation criteria
- blocking conditions
- explicit constraints
- propagation requirement

If Porky cannot produce a complete subtask description, Porky must not create the subtask yet.

A subtask with only a title is invalid delegation.

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

- `strategy` artifact
- `SYNTHESIS_READY` signal
- Phase-specific subtasks with complete descriptions
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

Required when Babe’s audit contains publishing risk:

- `excluded_claims`
- `safe_framing`

---

### Critical Fact / Brand Risk Gate

Before creating strategy, Porky must inspect Babe’s audit artifact for:

- `critical_flags`
- uncertainty
- inconsistencies
- `requires_human_confirmation`
- `unsafe_claims`
- brand positioning conflicts
- source conflicts
- publish-risk notes

If a risk affects publishable content, Porky must do one of two things:

### Option 1 — Block

Use this when the uncertainty prevents safe strategy.

```text
Status: BLOCKED — Strategy requires confirmation

Reason:
Babe identified an unresolved brand positioning or factual risk that affects publishable content.

Required confirmation:
<exact confirmation needed>
```

### Option 2 — Safe strategy

Use this when Porky can continue without using the risky claim.

The strategy must include:

- `excluded_claims`
- `safe_framing`
- `content_rules` that forbid risky claims

Porky must never build strategy angles, hooks, messaging pillars, or content rules on unresolved critical uncertainty.

---

### Allowed signals

Porky may emit:

- `SYNTHESIS_READY`
- `BLOCKED`
- `TIMEOUT_DETECTED`
- `RECOVERY_IN_PROGRESS`
- `RECOVERY_COMPLETE`

Porky must not emit:

- `AUDIT_READY`
- `CONTENT_SET_READY`
- `NOTION_SYNC_COMPLETE`
- `DELIVERY_COMPLETE`

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
- Final summary is posted on the parent issue

---

## Babe contract

### Role

Babe is the research and audit agent.

Babe transforms raw signals into a structured research artifact.

Babe is the first specialist agent in the sequential workflow.

---

### Responsibilities

Babe is responsible for:

- Running structured research through `audit.digital_presence`
- Auditing digital presence signals
- Analyzing X / Twitter
- Analyzing website and product surfaces
- Detecting patterns and inconsistencies
- Identifying opportunities
- Surfacing uncertainty and data gaps
- Identifying critical publishing or positioning risks
- Producing the `audit` artifact
- Emitting `AUDIT_READY`
- Posting the artifact and signal on the parent issue

---

### Forbidden actions

Babe must not:

- Create content drafts
- Define strategy
- Decide campaign priorities
- Persist anything to Notion
- Send Telegram summaries
- Orchestrate workflows
- Delegate to Hamm
- Delegate to Pumba
- Generate images

---

### Direct assignment execution rule

When Babe receives an assigned research task, Babe must execute the task directly.

Babe must not spend the run reading Paperclip reference documentation unless a required API or tool call fails and the reference is needed to recover.

Babe must not:

- perform broad coordination discovery
- inspect unrelated issues
- search for unassigned work
- read historical runs unless explicitly required
- loop on environment discovery when `parent_issue_id` and `subtask_issue_id` are already present

On assignment wake-up, Babe must:

1. Read the assigned issue.
2. Extract `parent_issue_id` and `subtask_issue_id`.
3. Validate required inputs.
4. Run `audit.digital_presence`.
5. Produce the audit artifact.
6. Post the artifact and `AUDIT_READY` signal on the parent task.
7. Optionally copy artifact and signal to the subtask.
8. Stop.

---

### Execution budget rule

Babe must keep research execution bounded.

If `audit.digital_presence` is slow, unavailable, or returns incomplete data, Babe must not loop indefinitely.

If the skill cannot complete within the current run, Babe must emit `BLOCKED` with the exact reason.

If data is incomplete but usable, Babe must continue with cautious language and document the limitation under uncertainty.

Babe should prefer a complete, structured, bounded audit over an unbounded research process.

Do not retry the same failed step more than once in the same run.

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

Both must be posted on the parent issue.

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
- `critical_flags`

If no critical flags exist, Babe must still include:

```text
critical_flags: []
```

---

### Critical flags

Babe must make publishing risks machine-readable.

Critical flags are required when research reveals any issue that could affect publishable content.

Examples:

- brand positioning conflict
- chain positioning uncertainty
- unsupported product claim
- unclear audience segment
- conflicting public signals
- outdated or incomplete external data
- claim requiring internal confirmation
- regulatory or financial-risk wording concern
- source conflict
- high-confidence gap that affects strategy

Required format:

```text
critical_flags:
- flag: <short name>
  severity: <low | medium | high>
  affects_publishing: <true | false>
  requires_human_confirmation: <true | false>
  description: <clear explanation>
  unsafe_claims:
    - <claim that should not be used>
  safe_framing:
    - <safe alternative framing>
```

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

- The full audit artifact is published on the parent issue
- All required sections exist
- `critical_flags` exists, even if empty
- `AUDIT_READY` is emitted on the parent issue
- The signal is valid
- The signal is optionally copied to the subtask for traceability

If Babe posts only to the subtask and not the parent issue, the workflow is incomplete.

---

## Hamm contract

### Role

Hamm is the content generation agent.

Hamm transforms Porky’s validated strategy artifact into structured, platform-native content.

Hamm is the second specialist agent in the sequential workflow.

---

### Responsibilities

Hamm is responsible for:

- Generating X posts
- Generating X threads
- Generating LinkedIn posts
- Following Porky’s strategy artifact
- Following `brand_constraints`
- Following `content_rules`
- Respecting `excluded_claims`
- Using `safe_framing` when provided
- Producing publication-ready content
- Producing structured content metadata
- Producing the `content_set` artifact
- Emitting `CONTENT_SET_READY`
- Posting the artifact and signal on the parent issue

---

### Forbidden actions

Hamm must not:

- Perform research
- Define strategy
- Persist content to Notion
- Send Telegram summaries
- Orchestrate workflows
- Delegate to Pumba
- Generate images
- Invent unsupported product claims
- Start before Porky explicitly delegates Hamm

---

### Required upstream state

Hamm may start only when:

- Babe audit artifact exists
- `AUDIT_READY` exists or has been reconciled
- Porky strategy artifact exists
- `SYNTHESIS_READY` exists or has been reconciled
- Porky explicitly delegated the content phase to Hamm

If these conditions are not met, Hamm must not generate content.

---

### Inputs

Hamm consumes:

- Strategy artifact
- Optional CEO brief
- Date range
- Content rules
- Platform mix
- Language requirements
- `excluded_claims`, if present
- `safe_framing`, if present
- Parent issue ID
- Subtask issue ID

---

### Outputs

Hamm produces:

- `content_set` artifact
- `CONTENT_SET_READY` signal

Both must be posted on the parent issue.

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
4. Solution or Piggy Wallet connection
5. CTA or closing insight

---

### Strategy compliance rule

Every content item must map to:

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
- `excluded_claims`
- `safe_framing`

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

### Financial and Web3 safety rules

Hamm must avoid:

- hype
- fear
- speculation
- investment advice
- get-rich messaging
- unsupported yield claims
- chain superiority claims
- unverified product claims
- regulatory-sensitive promises

Hamm must not say or imply:

```text
guaranteed protection
guaranteed returns
risk-free savings
best blockchain
fastest blockchain
official integration
powered by <chain>
```

unless Porky’s validated strategy explicitly permits it.

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

- The full `content_set` artifact is published on the parent issue
- All content items are present
- `item_count` matches actual items
- Schema is valid
- No duplicates exist
- Excluded claims are avoided
- Safe framing is followed when required
- `CONTENT_SET_READY` is emitted on the parent issue
- The signal is valid
- The signal is optionally copied to the subtask for traceability

If Hamm posts only to the subtask and not the parent issue, the workflow is incomplete.

---

## Pumba contract

### Role

Pumba is the Notion persistence and delivery agent.

Pumba turns validated artifacts into persisted, reviewable, distribution-ready outputs.

Pumba is the final specialist agent in the sequential workflow.

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
- Posting final artifacts and signals on the parent issue

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
- Start before Porky explicitly delegates Pumba
- Treat early wake-up as workflow failure

---

### Required upstream state

Pumba may start only when:

- Babe audit artifact exists
- `AUDIT_READY` exists or has been reconciled
- Hamm content set artifact exists
- `CONTENT_SET_READY` exists or has been reconciled
- Porky explicitly delegated the Notion + Delivery phase to Pumba

If these conditions are not met, Pumba must not persist content, must not send Telegram, and must not emit `NOTION_SYNC_COMPLETE` or `DELIVERY_COMPLETE`.

---

### No early execution rule

If Pumba wakes up early before Porky has explicitly delegated the Notion + Delivery phase:

- do not persist anything
- do not send Telegram
- do not emit `NOTION_SYNC_COMPLETE`
- do not emit `DELIVERY_COMPLETE`
- do not emit a noisy `BLOCKED` signal unless Porky explicitly delegated Pumba and required inputs are still missing

If the phase has not been explicitly delegated yet, Pumba should exit cleanly or leave a brief waiting note if the runtime requires a visible update.

This is not a workflow failure.

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
- Explicit delegation from Porky

---

### Outputs

Pumba produces or verifies:

- Notion Content Pipeline entries
- Research Babe page
- Telegram summary
- `notion_content_pipeline` artifact
- `telegram_delivery` artifact
- `NOTION_SYNC_COMPLETE` signal
- `DELIVERY_COMPLETE` signal

All final artifacts and signals must be posted on the parent issue.

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

```text
Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>
```

If date range is unavailable:

```text
Research Babe — <PARENT_ISSUE_ID>
```

The page must include:

- Research Summary
- Key Signals
- Inconsistencies
- Uncertainty / Data Gaps
- Opportunities
- Recommended Content Angles
- Critical Flags, if present
- Source / Workflow Metadata

---

### Output artifacts

Before emitting `NOTION_SYNC_COMPLETE`, Pumba must publish:

```text
ARTIFACT:
type: notion_content_pipeline
...
```

Before emitting `DELIVERY_COMPLETE`, Pumba must publish:

```text
ARTIFACT:
type: telegram_delivery
...
```

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
- `notion_content_pipeline` artifact is posted on the parent issue
- `NOTION_SYNC_COMPLETE` is emitted on the parent issue
- `telegram_delivery` artifact is posted on the parent issue
- `DELIVERY_COMPLETE` is emitted on the parent issue

If Pumba posts only to the subtask and not the parent issue, the workflow is incomplete.

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
3. Same artifact and signal are visible on the parent issue.

---

### Parent issue source of truth

The parent issue is the canonical workflow record.

Subtasks are useful for delegation and traceability, but final workflow state must be visible on the parent issue.

Agents may copy artifacts and signals to their subtasks, but the parent issue is mandatory.

---

### No role leakage

Agents must not perform work owned by other agents.

Examples:

- Babe must not write final content.
- Hamm must not perform research.
- Hamm must not redefine strategy.
- Pumba must not invent missing content.
- Pumba must not rewrite strategy.
- Porky must not skip validation.

---

### No premature downstream execution

Downstream agents must not run before upstream artifacts are valid.

Specifically:

- Hamm must not start before Babe’s audit and Porky’s strategy are valid.
- Pumba must not start before Babe’s audit and Hamm’s content set are valid.
- Porky must not create Hamm or Pumba subtasks before their required inputs exist.

---

### No title-only subtasks

Porky must never create a title-only subtask.

Every subtask must include enough context for the assigned agent to execute deterministically.

Minimum subtask context:

- parent issue ID
- phase
- assigned agent
- objective
- source parent brief
- required inputs
- required artifact
- required signal
- validation criteria
- constraints
- blocking conditions
- parent issue source-of-truth rule

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
- parent issue visibility
- subtask description completeness
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
- sequential delegation
- parent issue source of truth
- human-reviewable execution

This contract-based design is what makes the system reliable enough for repeatable content operations.

# Example Audit Artifact

This example shows what a valid `audit` artifact from Babe should look like.

The `audit` artifact is the structured research output used by Porky to synthesize strategy.

Babe must publish this artifact before emitting `AUDIT_READY`.

The artifact and signal must be posted on the original parent issue.

Subtask-only output is not enough for workflow completion.

---

## Purpose

The audit artifact captures:

- What was researched
- What signals were found
- What inconsistencies or gaps exist
- What opportunities were identified
- What content angles may be useful downstream
- What publishing risks or uncertain claims must be handled carefully

The audit artifact should not include final content drafts.

It should provide research context and directional insight for strategy synthesis.

Babe provides research inputs.

Porky decides how those inputs become strategy.

---

## Required behavior

Babe must execute the assigned research task directly.

Babe should not spend the run on broad Paperclip reference discovery, unrelated issue inspection, or environment exploration when the assigned task already includes:

- parent issue ID
- subtask issue ID
- research objective
- target entity
- workflow context

On assignment wake-up, Babe should:

1. Read the assigned issue.
2. Extract `parent_issue_id` and `subtask_issue_id`.
3. Validate required inputs.
4. Run `audit.digital_presence`.
5. Produce the audit artifact.
6. Post the artifact and `AUDIT_READY` on the parent issue.
7. Optionally copy the artifact and signal to the subtask.
8. Stop.

If research data is incomplete but usable, Babe should continue with cautious language and document the limitation under `uncertainty`.

If the research skill cannot complete, Babe should emit `BLOCKED` with the exact reason.

---

## Required sections

A valid audit artifact must include:

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

Do not omit the section.

---

## Critical flags

`critical_flags` make publishing risks machine-readable for Porky.

A critical flag is required when research reveals any issue that could affect strategy or publishable content.

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

Required critical flag format:

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

## Example

```text
ARTIFACT:
type: audit
issue: PIG-101
subtask_issue: PIG-102
entity: Piggy Wallet
timestamp: 2026-05-01T14:00:00Z
sections:

summary:
Based on available data, Piggy Wallet is positioned as a fintech and edtech product for families in emerging markets. The strongest visible themes are inflation protection, financial education for kids, and simple savings habits. The brand should continue emphasizing trust, clarity, and family-oriented financial empowerment.

key_signals:
- signal: Families in emerging markets face inflation pressure that can make saving feel unreliable.
  source: Brand positioning and product context.
  interpretation: Content should explain why saving behavior needs both education and protection.

- signal: Piggy Wallet aims to hide Web3 complexity behind a simple user experience.
  source: Product positioning.
  interpretation: Messaging should avoid crypto jargon and focus on familiar family outcomes.

- signal: Parents are likely the primary decision-makers, while kids are the educational beneficiaries.
  source: Audience model.
  interpretation: Content should speak to adults while keeping the mission centered on children’s financial habits.

- signal: The strongest emotional hook is not speculation or hype, but protecting a child’s future.
  source: Brand mission.
  interpretation: Content should be educational, calm, and trust-building.

inconsistencies:
- signal: Piggy Wallet uses Web3 infrastructure but aims for a Web2-like user experience.
  description: There is a potential messaging tension between the underlying technology and the desired simplicity for users.
  why_it_matters: Content should avoid making users feel they need to understand crypto in order to trust or use the product.

- signal: The product combines savings, education, and inflation protection.
  description: These themes can compete for attention if not organized clearly.
  why_it_matters: Strategy should group content into clear messaging pillars to avoid confusing the audience.

uncertainty:
- gap: Current live engagement data is not included in this artifact.
  impact: Content angles should be treated as strategic hypotheses until validated with platform performance data.

- gap: Exact product feature availability is not confirmed in this artifact.
  impact: Content should avoid specific feature claims unless they are already verified.

- gap: Audience segmentation by country or region is not fully specified.
  impact: Content should use broad emerging-market framing unless a specific geography is provided.

opportunities:
- opportunity: Create educational content explaining why money habits matter early.
  reasoning: This aligns with Piggy Wallet’s edtech mission and gives parents a non-technical reason to care.

- opportunity: Use inflation as a practical family problem rather than a fear-based topic.
  reasoning: This creates urgency without relying on panic or speculative messaging.

- opportunity: Frame Piggy Wallet as a bridge between saving, learning, and protection.
  reasoning: This helps connect the product’s multiple value propositions into one simple narrative.

- opportunity: Build weekly X threads around one core financial lesson.
  reasoning: Threads allow Piggy Wallet to explain complex ideas in a simple, progressive way.

recommended_content_angles:
- angle: “Saving is a habit, but protection is a system.”
  reasoning: Connects financial education with inflation-aware product positioning.

- angle: “What kids learn about money before they earn it.”
  reasoning: Strong educational angle for parents and families.

- angle: “Why a piggy bank is no longer enough in inflationary economies.”
  reasoning: Creates a clear bridge from traditional savings to Piggy Wallet’s modern approach.

- angle: “Financial education should feel simple, not technical.”
  reasoning: Supports the Web2-like UX promise while avoiding Web3 jargon.

- angle: “Parents do not need more financial complexity. They need safer defaults.”
  reasoning: Builds trust and speaks directly to the adult buyer/user.

critical_flags:
- flag: Product feature specificity
  severity: medium
  affects_publishing: true
  requires_human_confirmation: false
  description: Exact feature availability is not confirmed in this artifact, so downstream strategy and content should avoid specific feature claims unless separately validated.
  unsafe_claims:
    - Piggy Wallet already supports every savings feature mentioned in this campaign.
    - Piggy Wallet guarantees inflation protection.
    - Piggy Wallet provides risk-free savings.
  safe_framing:
    - Piggy Wallet is designed to help families build better savings habits.
    - Piggy Wallet focuses on financial education and savings protection.
    - Piggy Wallet uses modern infrastructure under the hood while keeping the experience simple.

- flag: Web3 complexity risk
  severity: low
  affects_publishing: true
  requires_human_confirmation: false
  description: Piggy Wallet uses Web3 infrastructure, but the audience should not be required to understand crypto concepts to understand the product.
  unsafe_claims:
    - Families need to understand crypto to use Piggy Wallet.
    - Parents should manage wallets, chains, and keys themselves.
  safe_framing:
    - Piggy Wallet abstracts technical complexity.
    - The product should be explained through simple family outcomes.
    - Web3 infrastructure should remain under the hood in general-audience content.
```

---

## Example with no critical flags

If no critical flags exist, the section must still appear:

```text
critical_flags: []
```

This is valid.

Missing the `critical_flags` section entirely is invalid.

---

## Matching signal

After publishing the full artifact on the parent issue, Babe may emit:

```text
SIGNAL:
type: AUDIT_READY
producer: Babe
issue: PIG-101
subtask_issue: PIG-102
status: COMPLETE
artifact: audit
timestamp: 2026-05-01T14:01:00Z
```

The signal must not be emitted before the artifact is visible.

Babe may also copy the same artifact and signal to the subtask for traceability.

The parent issue is mandatory.

---

## Validation checklist

A valid audit artifact should include:

- [ ] `ARTIFACT:` header
- [ ] `type: audit`
- [ ] Parent issue ID
- [ ] Babe subtask issue ID
- [ ] Entity name
- [ ] Timestamp
- [ ] Summary
- [ ] Key signals
- [ ] Inconsistencies
- [ ] Uncertainty / data gaps
- [ ] Opportunities
- [ ] Recommended content angles
- [ ] `critical_flags`
- [ ] `critical_flags` is present even if empty
- [ ] Critical flags are machine-readable when publishing risks exist
- [ ] Unsafe claims are listed when relevant
- [ ] Safe framing is listed when relevant
- [ ] No final content drafts
- [ ] Artifact is posted on the parent issue
- [ ] Signal emitted only after artifact is visible
- [ ] `AUDIT_READY` is posted on the parent issue

---

## Common invalid examples

Invalid audit artifacts include:

- Research notes without an `ARTIFACT:` block
- A summary without key signals
- Key signals without uncertainty
- Recommended content angles without reasoning
- Final posts or threads inside the audit
- Missing `critical_flags`
- Hiding publishing risks only in prose
- Unsupported claims presented as facts
- `AUDIT_READY` emitted before the artifact is visible
- Artifact posted only on the subtask and not the parent issue
- Missing parent issue ID
- Missing subtask issue ID

---

## Invalid example — missing critical_flags

```text
ARTIFACT:
type: audit
issue: PIG-101
subtask_issue: PIG-102
entity: Piggy Wallet
timestamp: 2026-05-01T14:00:00Z
sections:

summary:
Piggy Wallet focuses on financial education and family savings.

key_signals:
- signal: Piggy Wallet uses Web3 infrastructure.
  source: Product positioning.
  interpretation: Content can mention Web3.

inconsistencies:
- signal: Web3 infrastructure and simple UX.
  description: The messaging could become too technical.
  why_it_matters: Parents may not want technical explanations.

uncertainty:
- gap: Exact feature availability is not confirmed.
  impact: Content should avoid specific claims.

opportunities:
- opportunity: Explain financial education simply.
  reasoning: This fits the audience.

recommended_content_angles:
- angle: Money habits start early.
  reasoning: This fits Piggy Wallet’s mission.
```

Why invalid:

- `critical_flags` is missing.
- Potential publishing risks are described but not made machine-readable.
- Porky cannot reliably know which claims to exclude or safely frame.

---

## Invalid example — publishing risk hidden in prose

```text
uncertainty:
- gap: There is some uncertainty around whether Piggy Wallet should be described as chain-specific.
  impact: Be careful.
```

Why invalid:

- The risk affects publishable content.
- The unsafe claims are not listed.
- Safe framing is not provided.
- Porky cannot deterministically convert this into `excluded_claims` and `safe_framing`.

Better:

```text
critical_flags:
- flag: Chain positioning uncertainty
  severity: high
  affects_publishing: true
  requires_human_confirmation: true
  description: Available signals do not confirm that Piggy Wallet should be framed as chain-specific.
  unsafe_claims:
    - Piggy Wallet is Solana-native.
    - Piggy Wallet is powered by Solana.
    - Solana is the primary Piggy Wallet chain.
  safe_framing:
    - Piggy Wallet uses blockchain infrastructure under the hood.
    - Piggy Wallet abstracts crypto complexity for families.
    - Piggy Wallet helps families access stable digital savings.
```

---

## Notes

The audit artifact should help Porky synthesize strategy.

It should not replace the strategy artifact.

Babe provides research inputs.

Porky turns those inputs into strategy.

If Babe identifies risk, uncertainty, or incomplete evidence that could affect publishable content, it must be represented in `critical_flags`.

That is what allows Porky to create `excluded_claims` and `safe_framing` in the strategy artifact.

If Porky has to guess what is risky, the audit artifact is incomplete.

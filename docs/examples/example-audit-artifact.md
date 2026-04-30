# Example Audit Artifact

This example shows what a valid `audit` artifact from Babe should look like.

The `audit` artifact is the structured research output used by Porky to synthesize strategy.

Babe must publish this artifact before emitting `AUDIT_READY`.

---

## Purpose

The audit artifact captures:

- What was researched
- What signals were found
- What inconsistencies or gaps exist
- What opportunities were identified
- What content angles may be useful downstream

The audit artifact should not include final content drafts.

It should provide research context and directional insight for strategy synthesis.

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
```

---

## Matching signal

After publishing the full artifact, Babe may emit:

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
- [ ] No final content drafts
- [ ] Signal emitted only after artifact is visible

---

## Common invalid examples

Invalid audit artifacts include:

- Research notes without an `ARTIFACT:` block
- A summary without key signals
- Key signals without uncertainty
- Recommended content angles without reasoning
- Final posts or threads inside the audit
- `AUDIT_READY` emitted before the artifact is visible
- Missing parent issue ID
- Missing subtask issue ID

---

## Notes

The audit artifact should help Porky synthesize strategy.

It should not replace the strategy artifact.

Babe provides research inputs.

Porky decides how those inputs become strategy.

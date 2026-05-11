# Example Strategy Artifact

This example shows what a valid `strategy` artifact from Porky should look like.

The `strategy` artifact is the structured strategic output used by Hamm to generate a complete `content_set`.

Porky must publish this artifact before emitting `SYNTHESIS_READY`.

The artifact and signal must be posted on the original parent issue.

Subtask-only output is not enough for workflow completion.

---

## Purpose

The strategy artifact transforms Babe’s research into an actionable content plan.

It should define:

- Date range
- Expected number of content items
- Messaging pillars
- Content angles
- Weekly structure
- Brand constraints
- Content rules
- Excluded claims, when Babe identifies publishing risks
- Safe framing, when Babe identifies publishing risks

The strategy artifact should not include final post copy or full threads.

It should give Hamm enough structure to generate content without guessing.

---

## Required behavior

Porky must synthesize strategy only after Babe’s audit artifact is valid.

Before creating the strategy artifact, Porky must inspect Babe’s audit for:

- `critical_flags`
- uncertainty
- inconsistencies
- `requires_human_confirmation`
- `unsafe_claims`
- brand positioning conflicts
- source conflicts
- publishing-risk notes

If Babe’s audit contains critical publishing risks, Porky must either:

1. Block and request human confirmation, or
2. Continue only with safe strategy that excludes risky claims and provides safe framing.

Porky must not build strategy angles, hooks, messaging pillars, or content rules on unresolved critical uncertainty.

---

## Required fields

A valid strategy artifact must include:

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

## Critical Fact / Brand Risk Gate

The Critical Fact / Brand Risk Gate prevents unsafe claims from moving downstream.

If Babe flags a claim as unsafe, uncertain, unsupported, or requiring confirmation, Porky must not allow Hamm to use it.

Porky should convert Babe’s `critical_flags` into:

```text
excluded_claims:
- claim:
  reason:

safe_framing:
- framing:
  reason:
```

Hamm must then avoid all `excluded_claims` and use `safe_framing` when writing around uncertain topics.

---

## Example

```text
ARTIFACT:
type: strategy
issue: PIG-101
date_range: May 11–13, 2026
expected_items: 3

messaging_pillars:
- pillar: Financial education starts before kids earn money
  reasoning: Babe’s audit suggests that parents need simple ways to teach financial habits early, before children begin making independent financial decisions.

- pillar: Saving is a habit, protection is a system
  reasoning: The audit identifies inflation as a practical family problem. Piggy Wallet should connect healthy saving behavior with protection against value erosion, without implying guaranteed protection or risk-free outcomes.

- pillar: Simple user experience over financial complexity
  reasoning: Piggy Wallet uses modern financial infrastructure, but the audience should not need to understand technical concepts to trust the product.

content_angles:
- angle: Why kids need money habits before they need money
  source_from_audit: Recommended content angle around early money education.
  goal: Build educational trust with parents.

- angle: Why a traditional piggy bank is no longer enough
  source_from_audit: Audit identified inflation pressure as a key family concern.
  goal: Explain the need for modern savings habits and protection systems without making guaranteed protection claims.

- angle: Safer financial defaults for busy parents
  source_from_audit: Audit identified simplicity and trust as key positioning needs.
  goal: Position Piggy Wallet as helpful and understandable rather than complex or speculative.

weekly_structure:
- date: 2026-05-11
  recommended_format: X thread
  platform: X
  angle: Why a traditional piggy bank is no longer enough
  objective: Educate parents on inflation-aware saving.
  language: EN

- date: 2026-05-12
  recommended_format: X post
  platform: X
  angle: Why kids need money habits before they need money
  objective: Create a simple, shareable educational point.
  language: EN

- date: 2026-05-13
  recommended_format: LinkedIn post
  platform: LinkedIn
  angle: Safer financial defaults for busy parents
  objective: Build trust with adults and position Piggy Wallet as a family finance solution.
  language: EN

brand_constraints:
- constraint: Speak to adults and parents, not directly to children as buyers.
- constraint: Avoid hype, fear, speculation, or get-rich messaging.
- constraint: Avoid crypto jargon unless explicitly necessary.
- constraint: Emphasize simplicity, education, trust, and family outcomes.
- constraint: Do not invent product capabilities that are not verified.
- constraint: Keep the tone calm, clear, and mission-driven.
- constraint: Do not frame users as needing to understand crypto to use Piggy Wallet.
- constraint: Do not make guarantees about inflation protection, returns, safety, or risk elimination.

content_rules:
- rule: Generate exactly 3 content items.
- rule: Generate exactly 1 item per scheduled date.
- rule: Include 1 X thread, 1 X post, and 1 LinkedIn post.
- rule: The X thread must include 4–7 tweets.
- rule: Every content item must include title, platform, language, content_type, scheduled_date, vertical, version, week, and tags_and_mentions.
- rule: Posts must include body.
- rule: Threads must include thread_id and tweets.
- rule: Use version v1 unless otherwise specified.
- rule: Use ISO week format YYYY-WXX.
- rule: Do not duplicate titles or angles.
- rule: Do not use excluded claims.
- rule: Use safe framing when discussing inflation protection, blockchain infrastructure, or product capabilities.
- rule: Do not invent product features, integrations, partnerships, performance metrics, or technical architecture.

excluded_claims:
- claim: Piggy Wallet guarantees inflation protection.
  reason: Babe’s audit did not verify a guaranteed protection mechanism, and guarantee language creates financial-risk exposure.

- claim: Piggy Wallet provides risk-free savings.
  reason: Risk-free financial claims are unsupported and unsafe.

- claim: Families need to understand crypto to use Piggy Wallet.
  reason: Babe’s audit indicates the opposite: Piggy Wallet should abstract technical complexity.

- claim: Piggy Wallet is powered by a specific chain.
  reason: Chain-specific positioning was not confirmed in the audit and should not be used unless separately validated.

safe_framing:
- framing: Piggy Wallet is designed to help families build better savings habits.
  reason: This keeps the claim educational and mission-aligned without implying guaranteed outcomes.

- framing: Piggy Wallet focuses on financial education and savings protection.
  reason: This connects the product mission to the audience need without overpromising.

- framing: Piggy Wallet uses modern infrastructure under the hood while keeping the experience simple.
  reason: This explains the technology posture without requiring users to understand crypto.

- framing: In inflationary environments, families need tools and habits that help them think more intentionally about saving.
  reason: This discusses inflation pressure without making financial guarantees.
```

---

## Example without publishing risks

If Babe’s audit contains no critical publishing risks, Porky may omit `excluded_claims` and `safe_framing`.

However, if Babe includes:

```text
critical_flags: []
```

Porky should still confirm that no risky claims are being introduced by the strategy.

Example:

```text
ARTIFACT:
type: strategy
issue: PIG-101
date_range: May 11–13, 2026
expected_items: 3

messaging_pillars:
- pillar: Financial education starts early
  reasoning: Babe’s audit found that parents need simple educational language around money habits.

content_angles:
- angle: Money habits before the first paycheck
  source_from_audit: Babe recommended early financial education as a trust-building angle.
  goal: Build educational trust with parents.

weekly_structure:
- date: 2026-05-11
  recommended_format: X post
  platform: X
  angle: Money habits before the first paycheck
  objective: Share a simple educational insight.
  language: EN

brand_constraints:
- constraint: Speak to adults and parents.
- constraint: Avoid hype and unsupported claims.

content_rules:
- rule: Generate exactly 3 content items.
- rule: Use version v1.
- rule: Use ISO week format YYYY-WXX.
```

This is valid only if the audit truly contains no publishing risks and the strategy does not introduce new risky claims.

---

## Matching signal

After publishing the full strategy artifact on the parent issue, Porky may emit:

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
issue: PIG-101
status: COMPLETE
artifact: strategy
timestamp: 2026-05-01T14:10:00Z
```

The signal must not be emitted before the artifact is visible.

The parent issue is mandatory.

---

## Validation checklist

A valid strategy artifact should include:

- [ ] `ARTIFACT:` header
- [ ] `type: strategy`
- [ ] Parent issue ID
- [ ] Date range
- [ ] Expected item count
- [ ] Messaging pillars
- [ ] Reasoning for each pillar
- [ ] Content angles
- [ ] Source from audit for each angle
- [ ] Goal for each angle
- [ ] Weekly or date-based structure
- [ ] Recommended format per date
- [ ] Platform per date
- [ ] Language per date
- [ ] Brand constraints
- [ ] Content rules
- [ ] No final content drafts
- [ ] Signal emitted only after artifact is visible
- [ ] Strategy artifact is posted on the parent issue
- [ ] `SYNTHESIS_READY` is posted on the parent issue

When Babe’s audit contains critical publishing risks:

- [ ] `excluded_claims` exists
- [ ] `safe_framing` exists
- [ ] Unsafe claims from Babe’s audit are excluded
- [ ] Safe framing is specific enough for Hamm to use
- [ ] Content rules prevent risky claims from entering content
- [ ] Strategy does not build angles on unresolved critical uncertainty

---

## Common invalid examples

Invalid strategy artifacts include:

- A short summary without structured fields
- A vague instruction like “make engaging content”
- Missing date range
- Missing expected item count
- Missing content angles
- Missing brand constraints
- Missing content rules
- Strategy that does not map back to Babe’s audit
- Strategy that ignores Babe’s `critical_flags`
- Strategy that uses unresolved risky claims
- Missing `excluded_claims` when Babe’s audit identifies unsafe claims
- Missing `safe_framing` when Babe’s audit identifies unsafe claims
- Final post copy or full threads inside the strategy artifact
- `SYNTHESIS_READY` emitted before the strategy artifact is visible
- Strategy artifact posted only in a subtask and not on the parent issue

---

## Invalid example — vague strategy

```text
ARTIFACT:
type: strategy
issue: PIG-101
date_range: May 11–13, 2026
expected_items: 3

summary:
Create engaging content about financial education and savings for families.
```

Why invalid:

- No messaging pillars
- No content angles
- No weekly structure
- No brand constraints
- No content rules
- Hamm would have to guess what to write

---

## Invalid example — unresolved risky claim

```text
ARTIFACT:
type: strategy
issue: PIG-101
date_range: May 11–13, 2026
expected_items: 3

messaging_pillars:
- pillar: Guaranteed protection from inflation
  reasoning: Families care about inflation.

content_angles:
- angle: Piggy Wallet protects your savings from inflation, guaranteed.
  source_from_audit: Inflation pressure.
  goal: Create urgency.

weekly_structure:
- date: 2026-05-11
  recommended_format: X thread
  platform: X
  angle: Piggy Wallet protects your savings from inflation, guaranteed.
  objective: Drive awareness.
  language: EN

brand_constraints:
- constraint: Keep content simple.

content_rules:
- rule: Explain that Piggy Wallet guarantees inflation protection.
```

Why invalid:

- Uses guarantee language.
- Builds strategy on an unsupported financial claim.
- Does not include `excluded_claims`.
- Does not provide `safe_framing`.
- Creates unsafe downstream instructions for Hamm.

Better:

```text
excluded_claims:
- claim: Piggy Wallet guarantees inflation protection.
  reason: Guarantee language is unsupported and unsafe.

safe_framing:
- framing: Piggy Wallet is designed to help families build better savings habits and think more intentionally about protecting value.
  reason: This avoids guaranteed financial outcomes while preserving the educational angle.
```

---

## Invalid example — ignoring critical_flags

Babe audit includes:

```text
critical_flags:
- flag: Chain positioning uncertainty
  severity: high
  affects_publishing: true
  requires_human_confirmation: true
  description: Available data does not confirm chain-specific positioning.
  unsafe_claims:
    - Piggy Wallet is Solana-native.
    - Piggy Wallet is powered by Solana.
  safe_framing:
    - Piggy Wallet uses blockchain infrastructure under the hood.
```

Invalid strategy:

```text
content_angles:
- angle: Why Piggy Wallet is building on Solana
  source_from_audit: Chain positioning.
  goal: Explain technical differentiation.

content_rules:
- rule: Mention that Piggy Wallet is Solana-native.
```

Why invalid:

- Uses claims Babe explicitly marked as unsafe.
- Ignores `requires_human_confirmation`.
- Does not convert the critical flag into `excluded_claims`.
- Does not use the available safe framing.

---

## Notes

The strategy artifact should sit between research and content.

Babe provides research.

Porky turns that research into strategic direction.

Hamm turns the strategy into publication-ready content.

The strategy artifact should be specific enough for Hamm to execute, but not so detailed that it becomes final content.

If Babe identifies risk, uncertainty, or incomplete evidence that could affect publishable content, Porky must either block or convert that risk into `excluded_claims` and `safe_framing`.

If Hamm has to guess what is safe, the strategy artifact is incomplete.

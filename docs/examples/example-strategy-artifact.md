# Example Strategy Artifact

This example shows what a valid `strategy` artifact from Porky should look like.

The `strategy` artifact is the structured strategic output used by Hamm to generate a complete `content_set`.

Porky must publish this artifact before emitting `SYNTHESIS_READY`.

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

The strategy artifact should not include final post copy or full threads.

It should give Hamm enough structure to generate content without guessing.

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
  reasoning: The audit identifies inflation as a practical family problem. Piggy Wallet should connect healthy saving behavior with protection against value erosion.

- pillar: Simple user experience over financial complexity
  reasoning: Piggy Wallet uses modern financial infrastructure, but the audience should not need to understand technical concepts to trust the product.

content_angles:
- angle: Why kids need money habits before they need money
  source_from_audit: Recommended content angle around early money education.
  goal: Build educational trust with parents.

- angle: Why a traditional piggy bank is no longer enough
  source_from_audit: Audit identified inflation pressure as a key family concern.
  goal: Explain the need for modern savings protection.

- angle: Safer financial defaults for busy parents
  source_from_audit: Audit identified simplicity and trust as key positioning needs.
  goal: Position Piggy Wallet as helpful rather than complex.

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
```

---

## Matching signal

After publishing the full artifact, Porky may emit:

```text
SIGNAL:
type: SYNTHESIS_READY
producer: Porky
issue: PIG-101
status: COMPLETE
artifact: strategy
timestamp: 2026-05-01T14:10:00Z
```

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
- [ ] Brand constraints
- [ ] Content rules
- [ ] No final content drafts
- [ ] Signal emitted only after artifact is visible

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
- Final post copy or full threads inside the strategy artifact
- `SYNTHESIS_READY` emitted before the strategy artifact is visible

---

## Notes

The strategy artifact should sit between research and content.

Babe provides research.

Porky turns that research into strategic direction.

Hamm turns the strategy into publication-ready content.

The strategy artifact should be specific enough for Hamm to execute, but not so detailed that it becomes final content.

# Contributing to Open Agentic CMO

Thank you for your interest in contributing to Open Agentic CMO.

Open Agentic CMO is a deterministic, artifact-driven multi-agent system for content operations.

The project is built around strict contracts, validation gates, and human-reviewable outputs.

Contributions are welcome, but changes should preserve the core principle of the system:

> Agents should advance workflows by producing valid artifacts that pass validation, not by claiming work is done.

---

## Project scope

Open Agentic CMO currently focuses on the core workflow:

**Research → Strategy → Content → Notion → Delivery**

The current repository includes:

- Agent instructions
- Workflow documentation
- Signal contracts
- Artifact contracts
- Example artifacts
- Example signals
- JSON schemas
- Security guidance
- Environment variable placeholders

---

## Before contributing

Before making changes, please read:

- `README.md`
- `docs/01-overview.md`
- `docs/02-architecture.md`
- `docs/03-agent-contracts.md`
- `docs/04-signal-contract.md`
- `docs/05-artifact-contracts.md`
- `docs/06-orchestration-loop.md`
- `docs/07-notion-persistence.md`
- `docs/08-testing-e2e.md`
- `docs/09-failure-recovery.md`
- `SECURITY.md`

These files explain how the system is designed and what must not be broken.

---

## Contribution principles

All contributions should follow these principles:

1. Preserve deterministic execution.
2. Preserve artifact-before-signal behavior.
3. Preserve strict agent responsibilities.
4. Preserve validation gates.
5. Avoid introducing ambiguous workflow states.
6. Avoid adding unsupported future promises to documentation.
7. Keep public examples synthetic.
8. Never commit secrets or private data.

---

## Types of contributions

Useful contributions include:

- Improving documentation clarity
- Adding safe examples
- Improving JSON schemas
- Fixing inconsistencies between docs and agent contracts
- Adding validation checklists
- Improving failure recovery guidance
- Adding setup instructions
- Adding issue templates
- Improving security guidance

Potential future contributions may include:

- Validation scripts
- Local testing utilities
- Example workflows
- Notion setup helpers

---

## Agent contract changes

Agent instruction files live in:

```text
agents/porky/AGENTS.md
agents/babe/AGENTS.md
agents/hamm/AGENTS.md
agents/pumba/AGENTS.md
```

Changes to these files should be made carefully.

Agent contract changes may affect the full workflow.

If you change an agent contract, check whether you also need to update:

- `docs/03-agent-contracts.md`
- `docs/04-signal-contract.md`
- `docs/05-artifact-contracts.md`
- `docs/06-orchestration-loop.md`
- `docs/08-testing-e2e.md`
- relevant files in `docs/examples/`
- relevant files in `schemas/`

---

## Porky changes

Porky is the orchestrator and validation gatekeeper.

Be especially careful when changing Porky.

Do not weaken:

- signal validation
- artifact validation
- orchestration loop behavior
- idempotency rules
- downstream execution rules
- Research Babe validation
- final completion checks

Porky should never advance based on:

- summaries
- task status alone
- agent claims
- incomplete artifacts
- malformed signals

---

## Babe changes

Babe is the research and audit agent.

Babe should only produce structured research artifacts.

Do not make Babe responsible for:

- writing content
- defining strategy
- persisting to Notion
- sending Telegram messages
- orchestrating workflows

If you change Babe’s audit structure, update:

- `docs/05-artifact-contracts.md`
- `examples/example-audit-artifact.md` or `docs/examples/example-audit-artifact.md`
- `schemas/audit.schema.json`

---

## Hamm changes

Hamm is the content generation agent.

Hamm should only produce structured `content_set` artifacts.

Do not make Hamm responsible for:

- research
- strategy synthesis
- Notion persistence
- Telegram delivery
- orchestration

If you change Hamm’s content item structure, update:

- `docs/05-artifact-contracts.md`
- `examples/example-content-set-artifact.md` or `docs/examples/example-content-set-artifact.md`
- `schemas/content-set.schema.json`

---

## Pumba changes

Pumba is the Notion persistence and delivery agent.

Pumba should only validate, normalize, persist, and deliver.

Do not make Pumba responsible for:

- creating content
- defining strategy
- performing research
- orchestrating workflows
- inventing missing fields

Pumba must preserve:

- Content Pipeline persistence
- Research Babe persistence
- BODY-not-Notes rule
- deduplication behavior
- Telegram delivery validation

If you change Pumba behavior, update:

- `docs/07-notion-persistence.md`
- `docs/09-failure-recovery.md`
- relevant artifact contracts
- relevant examples

---

## Signal contract changes

Signals are workflow transition markers.

If you change signal names, producers, statuses, or artifact mappings, update:

- `docs/04-signal-contract.md`
- `docs/05-artifact-contracts.md`
- `docs/examples/example-signals.md`
- `schemas/signal.schema.json`
- agent instruction files that emit or validate those signals

Do not add new signals without explaining:

- who produces them
- what artifact they map to
- what status they use
- when they are valid
- how Porky should validate them

---

## Artifact contract changes

Artifacts are the source of truth.

If you change any artifact structure, update:

- `docs/05-artifact-contracts.md`
- the relevant example file
- the relevant JSON schema
- any agent that produces or consumes that artifact
- E2E testing documentation

Do not remove required fields unless the validation model is updated everywhere.

---

## Schema changes

Schemas live in:

```text
schemas/
```

When changing a schema:

- Keep examples valid.
- Keep required fields aligned with docs.
- Keep schema names aligned with artifact names.
- Avoid allowing ambiguous or unsupported fields.
- Update documentation if the schema changes the contract.

---

## Documentation changes

Documentation should describe what exists and is supported.

Avoid adding speculative roadmap items to core docs.

If you document future work, keep it clearly separated from implemented behavior.

Documentation should not include:

- real credentials
- private URLs
- production database IDs
- customer data
- private workflow logs
- sensitive screenshots

---

## Example changes

Examples should be safe, synthetic, and consistent with contracts.

Examples should use fake IDs such as:

```text
PIG-101
PIG-102
PIG-103
```

Do not include:

- real customer data
- real Notion database IDs
- real Telegram chat IDs
- real API responses
- real private issue links
- production screenshots with sensitive data

---

## Testing changes

If a change affects the workflow, run or document a controlled E2E test.

Recommended regression test:

- 3-day controlled E2E
- 1 X thread
- 1 X post
- 1 LinkedIn post
- Research Babe required
- Telegram summary required

See:

```text
docs/08-testing-e2e.md
```

---

## Pull request checklist

Before opening a pull request, verify:

- [ ] No secrets or private data are included.
- [ ] No real `.env` file is committed.
- [ ] Documentation matches the actual file structure.
- [ ] Agent contracts remain role-specific.
- [ ] Signals remain canonical and valid.
- [ ] Artifacts remain structured and validateable.
- [ ] Examples are synthetic and safe.
- [ ] Relevant schemas were updated if contracts changed.
- [ ] Relevant docs were updated if behavior changed.
- [ ] Security guidance was followed.

---

## Commit message style

Use clear, descriptive commit messages.

Good examples:

```text
Add signal JSON schema
Add audit artifact example
Update Notion persistence documentation
Fix content set schema validation
Add failure recovery documentation
```

Avoid vague messages:

```text
Update stuff
Fix things
Changes
Docs
```

---

## Security

Please read `SECURITY.md` before contributing.

Never commit:

- API keys
- access tokens
- bot tokens
- database IDs
- private URLs
- customer data
- private logs
- production screenshots with sensitive information

If you accidentally expose a secret, rotate it immediately.

---

## Review expectations

A good contribution should make the system more:

- deterministic
- explicit
- validateable
- recoverable
- auditable
- safe for downstream execution
- useful for human review

A contribution should not make the system more ambiguous or dependent on unstated assumptions.

---

## Summary

Open Agentic CMO is designed to be strict because strictness is what makes agentic workflows reliable.

When contributing, preserve the contract-based design:

- agents have narrow roles
- signals declare state
- artifacts prove work
- Porky validates transitions
- Pumba persists cleanly
- humans can review the output

Thank you for helping improve Open Agentic CMO.

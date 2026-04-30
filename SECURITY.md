# Security Policy

Security is important for Open Agentic CMO because the system may interact with third-party services such as Notion, Telegram, and future API providers.

This repository should never include real credentials, private workspace data, production database IDs, private issue data, or sensitive operational details.

---

## Supported project status

Open Agentic CMO is currently an open-source documentation and agent-contract repository.

The current public repository may include:

- Agent instructions
- Workflow documentation
- Signal contracts
- Artifact contracts
- Example artifacts
- Example signals
- JSON schemas
- Environment variable placeholders

The repository should not include:

- Real API keys
- Real Notion tokens
- Real Telegram bot tokens
- Real database IDs
- Private URLs
- Customer data
- Private analytics
- Internal team credentials
- Production workflow logs with sensitive information

---

## Environment variables

Use `.env.example` only for placeholder values.

Example:

```env
NOTION_API_KEY=your_notion_api_key_here
NOTION_CONTENT_DATABASE_ID=your_notion_content_database_id_here
NOTION_RESEARCH_DATABASE_ID=your_notion_research_database_id_here
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
TELEGRAM_CHAT_ID=your_telegram_chat_id_here
```

Never commit a real `.env` file.

Never commit credentials into:

- README files
- Documentation files
- Example files
- Agent instruction files
- JSON schemas
- Test files
- GitHub Issues
- Pull request descriptions

---

## Secret handling rules

Do not commit:

- API keys
- Access tokens
- Bot tokens
- OAuth secrets
- Database IDs from production workspaces
- Private Notion page URLs
- Telegram chat IDs from production environments
- Private repository URLs
- Internal workflow IDs
- Customer or user data

If a secret is accidentally committed:

1. Revoke the secret immediately.
2. Rotate the credential.
3. Remove the secret from the repository history if needed.
4. Open a security report or private maintainer note.
5. Do not continue using the exposed credential.

---

## Example data policy

Examples should use fake or placeholder data.

Safe examples:

- `PIG-101`
- `PIG-102`
- `your_notion_api_key_here`
- `your_telegram_bot_token_here`
- `2026-05-01T14:20:00Z`

Unsafe examples:

- Real Notion database IDs
- Real Telegram chat IDs
- Real customer names
- Real private issue IDs
- Real production URLs
- Real API responses containing private data

---

## Notion security

When using Notion integrations:

- Use the minimum required permissions.
- Avoid sharing production databases publicly.
- Do not expose internal database IDs in public examples.
- Use separate test databases for demos.
- Do not include private Notion page URLs in issues, examples, docs, or screenshots.

For open-source examples, use placeholder IDs only.

---

## Telegram security

When using Telegram integrations:

- Never commit bot tokens.
- Never commit private chat IDs.
- Use test bots for development.
- Rotate bot tokens if exposed.
- Avoid sending sensitive research or customer data into public or shared Telegram groups.

---

## Agent instruction safety

Agent instructions should not contain:

- Production credentials
- Private tool URLs
- Private workspace IDs
- Private team member information
- Customer-specific data
- Sensitive operational details

Agent files should define behavior, contracts, and responsibilities only.

---

## Reporting a vulnerability

If you discover a security issue, please do not open a public issue with sensitive details.

Instead, report it privately to the maintainers.

If a private security reporting channel is available on GitHub, use that.

If not, contact the repository owner directly through the appropriate private channel.

Include:

- A description of the issue
- Steps to reproduce, if relevant
- Potential impact
- Suggested fix, if known
- Whether any secrets or private data may have been exposed

---

## Public issue policy

Do not post sensitive information in public GitHub Issues.

Avoid including:

- API keys
- Tokens
- Database IDs
- Private URLs
- Screenshots containing secrets
- Customer data
- Internal workflow logs with sensitive content

If sensitive information is accidentally posted publicly, delete it if possible and rotate any exposed credentials immediately.

---

## Pull request policy

Before opening a pull request, check that it does not include:

- `.env`
- Real credentials
- Production IDs
- Private URLs
- Internal data
- Sensitive logs
- Customer data
- Generated files containing secrets

Pull requests that include secrets should not be merged.

---

## Recommended local files to ignore

The repository should avoid committing local configuration or secret files such as:

```text
.env
.env.local
.env.production
*.pem
*.key
secrets.json
credentials.json
```

If the project later adds code, update `.gitignore` accordingly.

---

## Security philosophy

Open Agentic CMO is designed around structured, auditable workflows.

Security follows the same principle:

> Sensitive data should never be implicit, hidden, or accidentally embedded in examples.

Keep credentials outside the repository.

Keep examples synthetic.

Keep public documentation safe to share.

---

## License

Security policy guidance applies to the project regardless of license.

Open Agentic CMO is released under the MIT License.

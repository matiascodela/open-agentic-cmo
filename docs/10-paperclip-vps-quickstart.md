# Paperclip VPS Quickstart

This guide explains how to install Paperclip on a VPS and use it as the control plane for Open Agentic CMO.

Paperclip is used as the orchestration environment where the Open Agentic CMO agents can be configured, assigned roles, and executed through an issue-driven workflow.

Open Agentic CMO currently uses the following core agents:

- Porky — Orchestrator / CEO-CMO
- Babe — Research / Audit
- Hamm — Content Lead
- Pumba — Notion Persistence / Delivery

The core workflow is:

```text
Research → Strategy → Content → Notion → Delivery
```

---

## Purpose

This quickstart is designed to help you:

1. Prepare a VPS.
2. Install Paperclip.
3. Access the Paperclip dashboard securely.
4. Create the Open Agentic CMO agent structure.
5. Add the `AGENTS.md` instructions.
6. Configure Notion and Telegram environment variables.
7. Run the controlled E2E test.

This guide focuses on a minimal, safe VPS setup for testing and operating the current Open Agentic CMO workflow.

---

## Important security note

Do not expose Paperclip publicly without authentication, firewall rules, or a secure tunnel.

For a first setup, the safest approach is:

- Run Paperclip bound to `127.0.0.1`
- Access it through an SSH tunnel
- Keep ports closed to the public internet unless you explicitly configure secure access

Never commit real credentials to this repository.

Never paste real API keys, Notion tokens, Telegram bot tokens, database IDs, or private URLs into public issues, examples, docs, or screenshots.

---

## Recommended VPS specs

For a small Open Agentic CMO test environment:

- Ubuntu 22.04 LTS or Ubuntu 24.04 LTS
- 2 vCPU
- 4 GB RAM minimum
- 40 GB disk
- SSH access
- A non-root user with sudo privileges

For heavier multi-agent workloads, use:

- 4+ vCPU
- 8+ GB RAM
- 80+ GB disk

---

## Prerequisites

You should have:

- A VPS provider account
- SSH access to the server
- Basic terminal knowledge
- A GitHub account
- Notion integration credentials
- Telegram bot token and chat ID, if using Telegram delivery
- Open Agentic CMO repo cloned locally or available on GitHub

---

## Official resources

Use the official documentation below when setting up or troubleshooting the stack.

### Paperclip

- Official Paperclip repository: https://github.com/paperclipai/paperclip
- Official Paperclip docs: https://paperclip.ing/docs
- Paperclip website: https://paperclip.ing
- Paperclip quickstart command: `npx paperclipai onboard --yes`

Paperclip is the control plane used to organize agents, goals, tasks, governance, budgets, and execution state.

### Notion

- Notion API documentation: https://developers.notion.com
- Notion API quickstart: https://developers.notion.com/guides/get-started/quick-start
- Notion integrations help: https://www.notion.com/help/create-integrations-with-the-notion-api

Use these docs to create the Notion integration, configure permissions, and connect databases.

### Telegram

- Telegram Bot API: https://core.telegram.org/bots/api
- Telegram bot tutorial: https://core.telegram.org/bots/tutorial
- BotFather: https://telegram.me/BotFather

Use these docs to create a Telegram bot, retrieve the bot token, and configure message delivery.

### Node.js and pnpm

- Node.js official website: https://nodejs.org
- NodeSource Node.js distributions: https://github.com/nodesource/distributions
- pnpm installation docs: https://pnpm.io/installation

Paperclip requires Node.js 20+ and pnpm when installing manually from source.

### Ubuntu, firewall, and Git

- Ubuntu Server firewall documentation: https://ubuntu.com/server/docs/how-to/security/firewalls
- Git installation docs: https://git-scm.com/install/linux

Use these docs for server security and Git setup.

---

## Step 1 — Create a VPS

Create a new VPS using Ubuntu.

Recommended OS:

```text
Ubuntu 22.04 LTS
```

or:

```text
Ubuntu 24.04 LTS
```

After creation, note:

- Server IP address
- SSH username
- SSH private key path

Example:

```text
IP: 203.0.113.10
User: root
SSH key: ~/.ssh/id_rsa
```

---

## Step 2 — Connect via SSH

From your local machine:

```bash
ssh root@YOUR_SERVER_IP
```

Example:

```bash
ssh root@203.0.113.10
```

If you use a custom private key:

```bash
ssh -i ~/.ssh/your_key root@YOUR_SERVER_IP
```

---

## Step 3 — Update the server

Run:

```bash
apt update && apt upgrade -y
```

Install basic utilities:

```bash
apt install -y curl git unzip ufw build-essential ca-certificates gnupg
```

---

## Step 4 — Create a non-root user

Create a deploy user:

```bash
adduser deploy
```

Add it to the sudo group:

```bash
usermod -aG sudo deploy
```

Copy SSH access from root to the new user:

```bash
rsync --archive --chown=deploy:deploy ~/.ssh /home/deploy
```

Switch to the new user:

```bash
su - deploy
```

From now on, use the `deploy` user when possible.

---

## Step 5 — Configure firewall

Official reference:

- Ubuntu Server firewall documentation: https://ubuntu.com/server/docs/how-to/security/firewalls

Enable OpenSSH:

```bash
sudo ufw allow OpenSSH
```

Enable the firewall:

```bash
sudo ufw enable
```

Check status:

```bash
sudo ufw status
```

For the safest first setup, do not open the Paperclip port publicly.

Paperclip should be accessed through an SSH tunnel.

---

## Step 6 — Install Node.js 20+

Official references:

- Node.js: https://nodejs.org
- NodeSource distributions: https://github.com/nodesource/distributions

Install Node.js 20.

One common Ubuntu setup path is:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify Node.js:

```bash
node -v
```

Expected:

```text
v20.x.x
```

Verify npm:

```bash
npm -v
```

---

## Step 7 — Install pnpm

Official reference:

- pnpm installation: https://pnpm.io/installation

Install pnpm globally:

```bash
sudo npm install -g pnpm
```

Verify:

```bash
pnpm -v
```

---

## Step 8 — Install Paperclip with quickstart

Official references:

- Paperclip repository: https://github.com/paperclipai/paperclip
- Paperclip docs: https://paperclip.ing/docs
- Paperclip website: https://paperclip.ing

Run:

```bash
npx paperclipai onboard --yes
```

This starts the Paperclip onboarding flow with default values.

The default quickstart may run Paperclip locally on port `3100`.

Expected local dashboard URL:

```text
http://127.0.0.1:3100
```

If the command asks interactive questions, choose safe local defaults for the first setup.

Recommended first-run mode:

- Bind host: `127.0.0.1`
- Port: `3100`
- Local/private access only
- Local storage/database defaults unless you know you need production-grade external services

---

## Step 9 — Keep Paperclip running

For quick testing, you can keep Paperclip running in the terminal.

For a longer-lived VPS setup, use a terminal multiplexer such as `tmux`.

Install tmux:

```bash
sudo apt install -y tmux
```

Create a session:

```bash
tmux new -s paperclip
```

Run Paperclip inside the session:

```bash
npx paperclipai onboard --yes
```

Detach from tmux:

```text
CTRL+b, then d
```

Reattach later:

```bash
tmux attach -t paperclip
```

---

## Step 10 — Access Paperclip securely through SSH tunnel

From your local machine, open an SSH tunnel:

```bash
ssh -L 3100:127.0.0.1:3100 deploy@YOUR_SERVER_IP
```

Example:

```bash
ssh -L 3100:127.0.0.1:3100 deploy@203.0.113.10
```

Then open in your local browser:

```text
http://127.0.0.1:3100
```

This lets you access the remote Paperclip dashboard without exposing the port publicly.

---

## Step 11 — Clone Open Agentic CMO repo

On the VPS:

```bash
git clone https://github.com/YOUR_ORG_OR_USER/open-agentic-cmo.git
cd open-agentic-cmo
```

If your repo is public:

```bash
git clone https://github.com/YOUR_ORG_OR_USER/open-agentic-cmo.git
```

If your repo is private, configure SSH keys or GitHub access first.

---

## Step 12 — Create `.env`

Copy the example environment file:

```bash
cp .env.example .env
```

Open it:

```bash
nano .env
```

Fill in your actual credentials:

```env
NOTION_API_KEY=your_real_notion_api_key
NOTION_CONTENT_DATABASE_ID=your_real_content_database_id
NOTION_RESEARCH_DATABASE_ID=your_real_research_database_id

TELEGRAM_BOT_TOKEN=your_real_telegram_bot_token
TELEGRAM_CHAT_ID=your_real_telegram_chat_id
```

Do not commit `.env`.

---

## Step 13 — Confirm `.env` is not tracked

Check Git status:

```bash
git status
```

If `.env` appears as an untracked file, do not add it.

If needed, add `.env` to `.gitignore`:

```bash
echo ".env" >> .gitignore
```

Also consider ignoring:

```bash
cat >> .gitignore <<'EOF'
.env
.env.local
.env.production
*.pem
*.key
secrets.json
credentials.json
EOF
```

---

## Step 14 — Create the Open Agentic CMO company in Paperclip

In the Paperclip dashboard, create a new company or workspace for:

```text
Open Agentic CMO
```

Suggested company description:

```text
A deterministic multi-agent CMO system for research, strategy, content operations, Notion persistence, and delivery.
```

Suggested business goal:

```text
Run a structured content marketing workflow from research to strategy, content generation, Notion persistence, and Telegram delivery using validated artifacts and signals.
```

---

## Step 15 — Create the agents

Create four agents in Paperclip.

### Porky

Name:

```text
Porky
```

Role:

```text
Orchestrator / CEO-CMO
```

Description:

```text
Owns orchestration, state reconciliation, strategy synthesis, signal validation, artifact validation, and workflow completion.
```

Instruction file:

```text
agents/porky/AGENTS.md
```

---

### Babe

Name:

```text
Babe
```

Role:

```text
Research / Audit Agent
```

Description:

```text
Produces structured audit artifacts from research signals and emits AUDIT_READY only after the full audit artifact exists.
```

Instruction file:

```text
agents/babe/AGENTS.md
```

---

### Hamm

Name:

```text
Hamm
```

Role:

```text
Content Lead Agent
```

Description:

```text
Turns Porky's strategy artifact into a structured content_set artifact with posts and threads.
```

Instruction file:

```text
agents/hamm/AGENTS.md
```

---

### Pumba

Name:

```text
Pumba
```

Role:

```text
Notion Persistence / Delivery Agent
```

Description:

```text
Persists content into Notion, creates or updates Research Babe, sends Telegram summary, and emits completion signals.
```

Instruction file:

```text
agents/pumba/AGENTS.md
```

---

## Step 16 — Add each AGENTS.md instruction

For each agent in Paperclip:

1. Open the agent configuration.
2. Paste the full content from the corresponding `AGENTS.md`.
3. Save the agent.
4. Confirm the role and responsibility are correct.

Files:

```text
agents/porky/AGENTS.md
agents/babe/AGENTS.md
agents/hamm/AGENTS.md
agents/pumba/AGENTS.md
```

---

## Step 17 — Configure secrets and integrations

Official integration references:

- Notion API docs: https://developers.notion.com
- Notion API quickstart: https://developers.notion.com/guides/get-started/quick-start
- Notion integrations help: https://www.notion.com/help/create-integrations-with-the-notion-api
- Telegram Bot API: https://core.telegram.org/bots/api
- Telegram bot tutorial: https://core.telegram.org/bots/tutorial
- BotFather: https://telegram.me/BotFather

Configure the required integrations for the environment where your agents will run.

Required for Notion persistence:

```text
NOTION_API_KEY
NOTION_CONTENT_DATABASE_ID
NOTION_RESEARCH_DATABASE_ID
```

Required for Telegram delivery:

```text
TELEGRAM_BOT_TOKEN
TELEGRAM_CHAT_ID
```

Make sure these are available to the relevant execution environment.

At minimum:

- Pumba needs Notion credentials.
- Pumba needs Telegram credentials.
- Porky needs access to issue/task state.
- Babe needs access to its research skill.
- Hamm needs access to its content generation skill.

---

## Step 18 — Set up Notion databases

Use the official Notion API docs when creating integrations and connecting databases:

- Notion API documentation: https://developers.notion.com
- Notion API quickstart: https://developers.notion.com/guides/get-started/quick-start

You need a Notion Content Pipeline database and a Research Babe destination.

### Content Pipeline required properties

The Content Pipeline should support:

```text
Title
Platform
Status
Scheduled Date
Vertical
Version
Week
Content Type
Thread ID
Thread Order
```

Minimum required fields:

```text
Title
Platform
Status
Scheduled Date
Vertical
Version
Week
Content Type
```

For threads:

```text
Thread ID
```

Content itself must be written into the Notion page body.

Do not use `Notes` for content body.

---

### Research Babe destination

Research Babe can be:

1. A dedicated Notion database
2. A section/page under the same workspace
3. A research archive database

The page title should follow:

```text
Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>
```

If no date range is available:

```text
Research Babe — <PARENT_ISSUE_ID>
```

The page body must include:

```text
Research Summary
Key Signals
Inconsistencies
Uncertainty / Data Gaps
Opportunities
Recommended Content Angles
Source / Workflow Metadata
```

---

## Step 19 — Run the controlled E2E test

Use the controlled E2E example:

```text
docs/examples/example-controlled-e2e-test.md
```

Create a new parent issue assigned to Porky.

Use this test scope:

```text
May 11, 2026 → May 13, 2026
```

Expected output:

```text
3 content items
1 X thread
1 X post
1 LinkedIn post
1 Research Babe page
1 Telegram summary
```

---

## Step 20 — Validate the run

After the test finishes, verify:

### Babe

- `ARTIFACT: type: audit` exists
- `AUDIT_READY` exists
- Signal appears after artifact

### Porky

- `ARTIFACT: type: strategy` exists
- `SYNTHESIS_READY` exists
- Validation gates were enforced

### Hamm

- `ARTIFACT: type: content_set` exists
- `CONTENT_SET_READY` exists
- 3 full content items exist
- Thread includes 4–7 tweets

### Pumba

- 3 content pages exist in Notion
- Research Babe page exists
- Content is in page body
- Content is not in Notes
- Version is populated
- Week is populated
- Content Type is populated
- Telegram summary was sent once
- `NOTION_SYNC_COMPLETE` exists
- `DELIVERY_COMPLETE` exists

---

## Step 21 — Troubleshooting

### Paperclip dashboard does not open

Check that Paperclip is running:

```bash
ps aux | grep paperclip
```

Check port:

```bash
ss -tulpn | grep 3100
```

If using SSH tunnel, make sure this is running locally:

```bash
ssh -L 3100:127.0.0.1:3100 deploy@YOUR_SERVER_IP
```

Then open:

```text
http://127.0.0.1:3100
```

---

### Port 3100 is already in use

Check process:

```bash
sudo lsof -i :3100
```

Kill the process if safe:

```bash
sudo kill -9 <PID>
```

Restart Paperclip.

---

### Permission issues

Make sure you are not mixing root and deploy user files.

Check ownership:

```bash
ls -la
```

If needed:

```bash
sudo chown -R deploy:deploy /home/deploy
```

---

### Node.js version is too old

Check:

```bash
node -v
```

If version is below Node.js 20, reinstall Node.js 20+.

---

### pnpm missing

Install:

```bash
sudo npm install -g pnpm
```

Verify:

```bash
pnpm -v
```

---

### Notion pages are created but content is in Notes

This is invalid.

Pumba must write:

- post body into Notion page body
- thread tweets into ordered page body blocks

Pumba must not store content in:

```text
Notes
metadata fields
fallback text fields
```

Use:

```text
docs/07-notion-persistence.md
docs/09-failure-recovery.md
```

---

### Research Babe page is missing

This is invalid.

`NOTION_SYNC_COMPLETE` requires both:

1. Content Pipeline entries
2. Research Babe page

Pumba must create or update:

```text
Research Babe — <PARENT_ISSUE_ID> — <DATE_RANGE>
```

---

### Signal exists but artifact is missing

This is invalid.

Example:

```text
CONTENT_SET_READY exists but content_set artifact is missing.
```

Porky must block and request the full artifact.

Use:

```text
docs/09-failure-recovery.md
```

---

## Security checklist

Before running real workflows:

- [ ] `.env` is not committed.
- [ ] Notion API key is private.
- [ ] Telegram bot token is private.
- [ ] Production database IDs are not in public docs.
- [ ] Paperclip is not publicly exposed without protection.
- [ ] SSH access is secured.
- [ ] Firewall is enabled.
- [ ] Examples use fake issue IDs.
- [ ] Screenshots do not expose private data.
- [ ] Test Notion databases are used before production databases.

---

## Recommended first validation run

Start with the controlled E2E test.

Do not start with a large campaign.

Recommended first run:

```text
3 days
1 X thread
1 X post
1 LinkedIn post
Research Babe required
Telegram summary required
```

Only run larger workflows after this passes.

---

## Summary

A safe Open Agentic CMO VPS setup should:

- Run Paperclip securely
- Keep credentials outside Git
- Use the four core agents
- Preserve strict agent contracts
- Persist content into Notion page bodies
- Persist Babe research into Research Babe
- Send Telegram summary once
- Validate the full workflow through Porky

The goal is not just to run agents.

The goal is to run a deterministic, recoverable, human-reviewable marketing operating system.

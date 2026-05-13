# Customer Feedback System — Claude Code Agent

An autonomous customer feedback pipeline built with Claude Code and CLAUDE.md. Reads multi-inbox emails, categorizes them with NLP, routes to teams, monitors SLAs, tracks bugs, scores feedback, and generates daily handoff reports — all on a cron schedule.

---

## What It Does

| Step | Action |
|------|--------|
| 📥 Ingest | Reads multiple inboxes (support@, bugs@, feedback@) via Gmail MCP |
| 🧠 Categorize | NLP keyword tagging — bugs, billing, features, security, questions |
| 📡 Route | Posts to the right Slack channel with priority score + sentiment |
| 🔁 Deduplicate | Checks resolved responses DB — sends cached reply if match found |
| ⏱️ SLA Monitor | If no action in 24hrs, emails customer + re-pings team |
| 🏆 Leaderboard | Scores product/team feedback weekly |
| 📋 EOD Report | Generates minutes of the day + overwrites Notion handoff doc nightly |

---

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed
- Node.js 18+
- MCP servers configured (see below)
- Notion workspace + Gmail account
- Slack workspace with bot token

---

## MCP Servers

Add the following to your `.mcp.json`:

```json
{
  "mcpServers": {
    "gmail": {
      "type": "http",
      "url": "https://gmailmcp.googleapis.com/mcp/v1"
    },
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/sse"
    },
    "notion": {
      "type": "http",
      "url": "https://mcp.notion.com/sse"
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "pipedream": {
      "type": "http",
      "url": "https://mcp.pipedream.net",
      "note": "Optional — for Outlook, Yahoo, or custom inboxes"
    }
  }
}
```

Authenticate each server by running:
```bash
claude mcp auth <server-name>
```

---

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/yourusername/customer-feedback-system
cd customer-feedback-system
```

**2. Configure parameters**

Open `CLAUDE.md` and update the config block at the top:

```yaml
INBOXES:
  - support@yourcompany.com
  - feedback@yourcompany.com
  - bugs@yourcompany.com

SLA_HOURS: 24
EOD_TIME: "18:00"
COMPANY_NAME: "YourCompany"

HANDOFF_DOC_ID: "your-notion-page-id"
MINUTES_DOC_ID: "your-notion-page-id"
BUG_DB_ID: "your-notion-database-id"
LEADERBOARD_DB_ID: "your-notion-database-id"
RESOLVED_RESPONSES_DB: "your-notion-database-id"

SLACK_CHANNELS:
  engineering: "#eng-tickets"
  product: "#product-feedback"
  billing: "#billing-ops"
  security: "#security-alerts"
  support: "#customer-support"
```

**3. Add cron jobs**
```cron
# Process inbox every 15 minutes
*/15 * * * *  claude --headless "process inbox" --project /path/to/project

# SLA check every hour
0 * * * *     claude --headless "check SLA breaches" --project /path/to/project

# EOD report at 6pm
0 18 * * *    claude --headless "run EOD report" --project /path/to/project
```

---

## Configurable Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `SLA_HOURS` | `24` | Hours before escalation triggers |
| `EOD_TIME` | `18:00` | Time to run end-of-day jobs |
| `TEAM_ROUTING` | See CLAUDE.md | Keywords mapped to teams |
| `DUPLICATE_THRESHOLD` | `80%` | Similarity score for auto-resolution |
| `LEADERBOARD_RESET` | Weekly | How often leaderboard resets |
| `AUTO_REPLY_DELAY_MSG` | See CLAUDE.md | Customer acknowledgement template |
| `OVERDUE_REPLY_MSG` | See CLAUDE.md | SLA breach customer message |

---

## Project Structure

```
├── CLAUDE.md              # Agent instructions, routing logic, all config
├── .mcp.json              # MCP server definitions
├── .env                   # API keys and secrets (not committed)
├── .env.example           # Template for environment variables
└── README.md
```

---

## Guardrails

- Security-tagged emails are **never** auto-resolved — always routed to a human
- Emails are never sent without being logged first
- Bug history is never overwritten — only status fields update
- Handoff doc is **overwritten** nightly (not appended)
- Minutes doc is **appended** nightly (not overwritten)
- If any MCP write fails — retries 3x, then alerts `#eng-tickets`

---

## Related Project

This repo is part of a broader CLAUDE.md experiments series. The companion project uses **Playwright MCP** to automate job applications — controlling Chrome, filtering roles, and submitting forms autonomously.

→ [claude-job-agent](https://github.com/yourusername/claude-job-agent)

---

## Contributing

PRs welcome — especially for:
- Additional MCP integrations (Zendesk, Intercom, HubSpot)
- Improved NLP categorization prompts
- Multi-language support for email parsing

---

## License

MIT

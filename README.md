# agent-trust

Community trust network for agent payment endpoints. Downloads nightly trust scores, provides pre-transaction verification, and accepts anonymous outcome reports.

## Security & Privacy

- Trust database is stored at `data/trust.json`, mode `0600`
- Dashboard binds to `127.0.0.1:18921` — not reachable from the network
- Endpoint URLs are SHA-256 hashed — full URLs never leave your machine
- Transaction amounts are bucketed into ranges before reporting (e.g. `0.01-0.10`) — exact amounts are never sent
- No wallet addresses, transaction details, or user identity is ever transmitted
- Reporting is opt-in; by default the skill only downloads, never sends

See [TECHNICAL.md](TECHNICAL.md) for the full security model, data model, and score formula.

## Install

```bash
clawhub install agent-trust
```

Or manually:

```bash
git clone https://github.com/agent-budget/agent-trust.git ~/.openclaw/skills/agent-trust
```

## Use

Ask your agent:
- "Is this endpoint trustworthy?" (before paying any service)
- "Report that last transaction as a failure"
- "Show me the trust dashboard"

Or use the CLI directly:

```bash
# Check an endpoint before paying
~/.openclaw/skills/agent-trust/scripts/check-endpoint.sh https://api.example.com/data

# Start visual dashboard at http://127.0.0.1:18921
~/.openclaw/skills/agent-trust/scripts/dashboard.sh start

# Force-refresh trust database
~/.openclaw/skills/agent-trust/scripts/sync-trust-db.sh --force
```

## Configuration

In `~/.openclaw/openclaw.json` or via the dashboard Settings tab:

```json
{
  "skills": {
    "agent-trust": {
      "enabled": true,
      "config": {
        "trust_db_url": "https://api.agent-trust.net/trust.json",
        "report_endpoint": "https://api.agent-trust.net/reports",
        "sync_interval_hours": 24,
        "participate_in_network": false,
        "auto_positive_signals": false
      }
    }
  }
}
```

## Requirements

- Node.js 18+
- OpenClaw 2026.3.x+
- No npm dependencies

## Project Structure

```
agent-trust/
├── SKILL.md              # Agent-facing instructions (what the LLM reads)
├── README.md             # This file
├── TECHNICAL.md          # Architecture, security model, score formula
├── server/
│   ├── trust-db.js       # Data layer: lookup, scoring, config
│   ├── reporter.js       # Signal construction, queue, submission
│   ├── server.js         # HTTP server and API endpoints
│   └── index.html        # Dashboard UI (no build step, no CDN)
├── scripts/
│   ├── check-endpoint.sh
│   ├── report.sh
│   ├── sync-trust-db.sh
│   ├── status.sh
│   └── dashboard.sh
└── data/                 # Created at runtime, all mode 0600
    ├── trust.json
    ├── pending-reports.jsonl
    └── config.json
```

## Tests

```bash
node --test test/test.js
```

## License

MIT

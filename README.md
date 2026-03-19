# fraud-filter

Community transaction outcome report network for agent payment endpoints. Downloads nightly satisfaction scores, provides pre-transaction verification, and submits anonymous outcome reports.

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
clawhub install fraud-filter
```

Or manually:

```bash
git clone https://github.com/fraud-filter/fraud-filter.git ~/.openclaw/skills/fraud-filter
```

## Use

Ask your agent:
- "Check this endpoint before I pay" (before paying any service)
- "Report that last transaction as a failure"
- "Show me the trust dashboard"

Or use the CLI directly:

```bash
# Check an endpoint before paying
~/.openclaw/skills/fraud-filter/scripts/check-endpoint.sh https://api.example.com/data

# Start visual dashboard at http://127.0.0.1:18921
~/.openclaw/skills/fraud-filter/scripts/dashboard.sh start

# Force-refresh trust database
~/.openclaw/skills/fraud-filter/scripts/sync-trust-db.sh --force
```

## Configuration

In `~/.openclaw/openclaw.json` or via the dashboard Settings tab:

```json
{
  "skills": {
    "fraud-filter": {
      "enabled": true,
      "config": {
        "trust_db_url":    "https://api.fraud-filter.com/trust.json",
        "report_endpoint": "https://api.fraud-filter.com/reports",
        "sync_interval_hours": 24,
        "participate_in_network": false,
        "on_block":   "block",
        "on_caution": "warn"
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
fraud-filter/
├── SKILL.md              # Agent-facing instructions (what the LLM reads)
├── README.md             # This file
├── TECHNICAL.md          # Architecture, security model, score formula
├── server/
│   ├── trust-db.js       # Data layer: lookup, scoring, config
│   ├── reporter.js       # Signal construction, queue, submission
│   ├── server.js         # HTTP server and API endpoints
│   └── index.html        # Dashboard UI (no build step, no CDN)
├── hooks/
│   ├── before-payment.sh  # PreToolUse — checks endpoint before payment fires
│   └── after-payment.sh   # PostToolUse — auto-reports empty/garbage responses
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

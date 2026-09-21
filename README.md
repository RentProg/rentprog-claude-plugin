# RentProg plugin for Claude Code (and any MCP client)

Read-only access to your **RentProg** rental-fleet data for AI agents through MCP, plus five ready-made skills:
`/rentprog:morning-brief`, `/rentprog:debtors`, `/rentprog:fleet-load`, `/rentprog:client-dossier`, `/rentprog:period-summary`.

The plugin talks to RentProg's MCP endpoint (`POST /mcp`, Streamable HTTP) with your **personal agent key**. The key acts on your behalf: the agent sees exactly the branches, records and fields you see in RentProg — nothing more. Wave 1 is read-only: no tool creates or changes anything.

## Requirements

- A RentProg account whose plan includes the **“MCP for AI agents”** service. If the *AI agent keys* tab in your profile shows “not included in your plan”, ask RentProg support to enable it. Pricing: https://rentprog.com/en/tariffs
- Claude Code 2.x (plugins with `userConfig`), or any MCP client that lets you set a request header — see “Other MCP clients”. Claude Cowork support is not confirmed yet — see “Known limitations”.

## 1. Issue a key

RentProg → your **Profile** → tab **AI agent keys** → **Create key**. Give it a name (e.g. “Claude Code — laptop”) and copy the key: it starts with `rpa_` and is shown **once**. Up to 5 active keys per user; each key is valid for one year and can be revoked from the same tab.

Partners and agents (external roles) cannot issue keys.

## 2. Install the plugin

```
/plugin marketplace add RentProg/rentprog-claude-plugin
/plugin install rentprog@rentprog
```

Claude Code asks for two settings:

| Setting | Value |
| --- | --- |
| RentProg personal agent key | your `rpa_…` key |
| RentProg MCP endpoint | leave the default `https://rentprog.net/mcp` |

Other endpoints: staging `https://rentprog.pro/mcp`; a local development stand `http://localhost:3155/mcp`.

Check the connection: `/mcp` should list the `rentprog` server with 12–14 tools (depending on your role and whether CRM is enabled), and `company_reference` should return your branches.

## 3. Use it

Ask in your own language. Examples:

- “What's on today?” → `/rentprog:morning-brief`
- “Who owes us more than 5 days?” → `/rentprog:debtors`
- “Which cars have been idle for a week?” → `/rentprog:fleet-load`
- “Tell me about client Ivanov before I call him” → `/rentprog:client-dossier`
- “How much did we earn last week?” → `/rentprog:period-summary`

Every answer comes from a tool call; every record carries a `ui_url` you can open in RentProg.

### Daily report on a schedule

In Claude Code you can schedule the brief, e.g. every weekday at 08:30:

```
/loop 08:30 weekdays /rentprog:morning-brief
```

(or use your OS scheduler to run `claude -p "/rentprog:morning-brief"` and send the output to Telegram/email).

## What the agent can see

| Role in RentProg | Scope | Tools | Fields |
| --- | --- | --- | --- |
| superadmin, admin | whole group | all 14 | everything, incl. purchase/sale prices and money report |
| guest | whole group | all 14 | money incl. finance; **no personal data of individual clients** (name, phone, email, birthday, tax id, balance) — legal entities are shown in full |
| manager, user | own branch (whole group with “can change branch”) | all except `money_report` | operational booking sums; **no car cost data** |
| partner, agent | — | keys are not issued | — |

`crm_inbox` appears only when the CRM add-on is active and the user has CRM access.

## Tools

`company_reference` · `get_card` · `search_bookings` · `search_clients` · `search_cars` · `schedule` · `availability_quote` · `receivables` · `fleet_status` · `client_dossier` · `money_report` · `fines` · `crm_inbox` · `recent_changes`

Limits per key: 60 calls/minute, 1000 calls/hour. Windows: schedule and recent_changes ≤ 31 days, money_report and fines ≤ 366 days; pages ≤ 50 items.

## Other MCP clients

The plugin packages the server for Claude Code, but the server itself is a plain MCP endpoint over
HTTP with a bearer token. Any client that lets you set a request header can use it with the same
key — no server-side setup, no separate account.

### Cursor

Create `.cursor/mcp.json` in your project (or `~/.cursor/mcp.json` for all projects):

```json
{
  "mcpServers": {
    "rentprog": {
      "url": "https://rentprog.net/mcp",
      "headers": { "Authorization": "Bearer rpa_your_key" }
    }
  }
}
```

Reload the MCP servers in Cursor settings; `rentprog` appears with the same tools your role sees in
Claude Code. The five skills are Claude Code slash commands and do not carry over — in Cursor just
ask in plain language, the tool descriptions are self-sufficient.

Keep the file out of version control: it contains your key. A revoked key stops working
immediately, so revoking in RentProg is enough if the file leaks.

### ChatGPT

Not supported yet. Custom connectors there accept either no authentication or OAuth 2.1 with
dynamic client registration — a personal bearer key cannot be entered. OAuth support is planned as
a separate piece of work, not a setting.

## Known limitations

- Read-only. Creating bookings, sending messages to clients and writing OCR results are planned for wave 2.
- Claude Cowork: plugin installation with `userConfig` is not yet confirmed in Cowork; use Claude Code for now.
- Cash balances in `money_report` are cashbox balances (card-to-card, terminal, bank); employees' personal cash accounts are not included.

## Security

Your key is stored by Claude Code as a sensitive plugin setting and sent only to the configured RentProg endpoint as `Authorization: Bearer …`. Revoke a key any time in RentProg → Profile → AI agent keys. RentProg logs every tool call (who, which tool, how long, outcome) without arguments or data.

## License

MIT — see [LICENSE](LICENSE).

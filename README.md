# RentProg plugin for Claude Code (and any MCP client)

Your **RentProg** rental-fleet data and actions for AI agents through MCP — CRM, fleet and bookings, money, staff — plus five ready-made skills:
`/rentprog:morning-brief`, `/rentprog:debtors`, `/rentprog:fleet-load`, `/rentprog:client-dossier`, `/rentprog:period-summary`.

The plugin talks to RentProg's MCP endpoint (`POST /mcp`, Streamable HTTP) with your **personal agent key**. The key acts on your behalf: the agent sees exactly the branches, records and fields you see in RentProg — nothing more. For each area you choose what the key may do: nothing, read, or change data — with your approval, with a preview, or immediately. A new key only reads.

## Requirements

- A RentProg account whose plan includes the **“MCP for AI agents”** service. If the *AI agent keys* tab in your profile shows “not included in your plan”, ask RentProg support to enable it. Pricing: https://rentprog.com/en/tariffs
- Claude Code 2.x (plugins with `userConfig`), or any MCP client that lets you set a request header — see “Other MCP clients”. Claude Cowork support is not confirmed yet — see “Known limitations”.

## 1. Issue a key

RentProg → your **Profile** → tab **AI agent keys** → **Create key**. Give it a name (e.g. “Claude Code — laptop”), choose the access level for each area (see “Access by area”) and copy the key: it starts with `rpa_` and is shown **once**. The same tab shows the MCP server address for your company (it depends on your country and region) — you will need it in step 2. Up to 5 active keys per user; each key is valid for one year and can be revoked from the same tab — it stops working immediately.

Partners and agents (external roles) cannot issue keys.

Keys issued before access by area was introduced are marked “Read only, previous format”: they see the previous 16 read tools without CRM. Reissue the key to get the full catalog.

## 2. Install the plugin

```
/plugin marketplace add RentProg/rentprog-claude-plugin
/plugin install rentprog@rentprog
```

Claude Code asks for two settings:

| Setting | Value |
| --- | --- |
| RentProg personal agent key | your `rpa_…` key |
| RentProg MCP endpoint | the MCP server address shown on the same **AI agent keys** tab — it depends on your company's country and region |

Check the connection: `/mcp` should list the `rentprog` server, `company_reference` should return your branches, and `whoami` shows the key's effective level in each area. The number of tools depends on your role and the key's levels. To set a new key, run `/plugin configure rentprog` and reconnect the server in `/mcp`.

## 3. Use it

Ask in your own language. Examples:

- “What's on today?” → `/rentprog:morning-brief`
- “Who owes us more than 5 days?” → `/rentprog:debtors`
- “Which cars have been idle for a week?” → `/rentprog:fleet-load`
- “Tell me about client Ivanov before I call him” → `/rentprog:client-dossier`
- “How much did we earn last week?” → `/rentprog:period-summary`
- “Which leads are waiting for a reply?”, “Extend booking 1234 by two days”, “Record a 5,000 cash payment for booking 1234” — plain language works too; changes follow the key's levels.

Every answer comes from a tool call; every record carries a `ui_url` you can open in RentProg.

### Daily report on a schedule

In Claude Code you can schedule the brief, e.g. every weekday at 08:30:

```
/loop 08:30 weekdays /rentprog:morning-brief
```

(or use your OS scheduler to run `claude -p "/rentprog:morning-brief"` and send the output to Telegram/email).

## Access by area

| Area | What it includes |
| --- | --- |
| CRM | leads, pipeline, conversations with clients (including Teletype), CRM tasks |
| Fleet and bookings | bookings, cars, clients, schedule, fines, booking documents, inventory and add-ons, rent-to-own, telematics, marketplaces, taxi, tasks, to-dos and checklists |
| Money | payments and cash desks, transfers, debts, invoices, deposits, investors, contractors, money for a period, fleet economics, staff performance, suspicious operations |
| Staff | employees and agents, their balances, shifts and shift templates, ratings |

| Level | What the agent can do |
| --- | --- |
| No access | does not see the area |
| Read | only looks at data |
| Write with approval | prepares a change that runs only after you approve it in RentProg |
| Write with preview | first gets the result of the change (e.g. the reply text to a client), then confirms it itself within 10 minutes |
| Write immediately | makes the change at once |

A new key reads CRM, fleet and money; staff is “No access”. Levels above what your role allows are not offered, and if your rights are reduced later, the key follows the new rights immediately. We recommend “Write with approval” for money and “Write with preview” for CRM.

## Changes through the agent

The agent changes data the same way an employee does on the RentProg screens, with the same checks: what the screen does not let your role do, the agent will not do either.

- **CRM:** reply to a client (including Teletype), create and edit a lead, move it through the pipeline, link a booking or contact, save a file from the conversation, notes and tasks, archive, convert to a client, Avito blacklist.
- **Fleet and bookings:** create and edit a booking, change its status, hand over, take back, extend; add-ons, damages, photos; clients and their files; cars, prices, maintenance, inspections, repairs; fines; inventory; rent-to-own; a template message to the client; marketplace decisions; telematics and taxi; tasks, to-dos and checklists.
- **Money:** payments (create, post, cancel), booking payment, transfers between employees, debts, invoices, fine payment, investors, contractors.
- **Staff:** employees and agents, balances, shifts, ratings.

The agent does not create documents — a contract is put together in RentProg; the agent sends existing booking documents and requests a signature (DocuSeal, OkiDoki).

With “Write with approval” a prepared change waits in RentProg → **Awaiting approval** (menu item and a notification in the bell); without a decision it expires in 24 hours. A client signature passed by the agent always waits for approval. Some actions go out into the world and cannot be undone — messages to clients, the Avito blacklist, sending documents, commands to a car, marketplace decisions, taxi operations, paid client checks; some of them run in the background (`accepted`), and the agent learns the outcome with `operation_status`. Every change carries an `idempotency_key`, so a repeated call does not do the work twice.

Everything the agent changed is marked “via agent” in cards, payments and booking history.

## What the agent can see by role

Key levels cap the agent from above; the employee's role also limits the data scope and fields.

| Role in RentProg | Scope | Key levels | Fields and tools |
| --- | --- | --- | --- |
| superadmin | whole group | up to “Write immediately” in every area | everything, incl. car cost and the money report |
| admin | own branch — whole group only with “can change branch” | up to “Write immediately” in every area | everything, incl. car cost and the money report |
| manager | own branch — whole group only with “can change branch” | up to “Write immediately” in every area | operational booking sums; **no car cost and no money report** |
| user | own branch — whole group only with “can change branch” | up to “Write immediately”, except Staff | operational booking sums; **no car cost, money report, fleet economics, staff performance or suspicious operations** |
| guest | whole group | “Read” only, except Staff | money in full; **no personal data of individual clients** (name, phone, email, birthday, tax id, balance) — companies in full; CRM as on the CRM screen, with contacts |
| partner, agent | — | keys are not issued | — |

The CRM area is available only if CRM is enabled for your company and you have CRM access, as on the CRM screen.

## Tools

56 read tools and 75 write tools across the four areas; your key sees the ones its levels and your role allow (`/mcp` lists them, `whoami` shows the levels). The read core: `company_reference` · `get_card` · `search_bookings` · `search_clients` · `search_cars` · `schedule` · `availability_quote` · `receivables` · `fleet_status` · `client_dossier` · `money_report` · `fleet_economics` · `staff_performance` · `anomalies` · `fines` · `recent_changes` · `crm_leads` · `crm_lead` · `crm_messages` · `crm_tasks` · `operation_status`.

Limits per key: reads — 60 calls/minute and 1000/hour; changes — 20/minute and 300/hour. Windows: `schedule` and `recent_changes` ≤ 31 days; `money_report`, `fines`, `fleet_economics`, `staff_performance` and `anomalies` ≤ 366 days; pages ≤ 50 items.

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
      "url": "MCP_address_from_your_profile",
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

### Codex

Codex connects the server the same way as Cursor — the key goes in an environment variable:

```bash
export RENTPROG_API_KEY=rpa_your_key
codex mcp add rentprog --url <MCP address from your RentProg profile> --bearer-token-env-var RENTPROG_API_KEY
```

### ChatGPT and other agents with a terminal: the CLI

ChatGPT agent mode cannot connect an MCP server with a key; it — and any agent that runs shell commands —
uses the RentProg CLI, [`@rentprog/cli`](https://github.com/RentProg/rentprog-cli). Every tool of this
plugin is a CLI command, with the same key and the same access.

When you create a key, the key window also shows a **command for agents with a terminal** — it already
contains the key and the address of your region. Give it to the agent (Node.js 20.18+ is required;
ChatGPT has it), then the agent works with `npx -y @rentprog/cli@0 tools`, `… help <tool>` and
`… <tool> [flags]`. Writes follow the key's levels: a preview is applied only after confirmation
(`--yes`, a `y` answer in a terminal, or the continuation command the CLI prints); `--wait` waits for your
decision on an approval.

## Known limitations

- The agent does not create documents (contracts are put together in RentProg); it sends existing ones.
- Claude Cowork: plugin installation with `userConfig` is not yet confirmed in Cowork; use Claude Code for now.
- Cash balances in `money_report` are cashbox balances (card-to-card, terminal, bank); employees' personal cash accounts are not included.

## Security

Your key is stored by Claude Code as a sensitive plugin setting and sent only to the configured RentProg endpoint as `Authorization: Bearer …`. Revoke a key any time in RentProg → Profile → AI agent keys. RentProg logs every tool call (who, which tool, how long, outcome) without arguments or data.

**Where the data goes.** Everything the key can see, including your clients' personal data, is sent by the agent to the AI service you connect. Your company chooses that service and, as the data controller, is responsible for sharing data with it. If the agent does not need clients' personal data, issue the key to an employee with the guest role and set CRM to “No access”.

Text written by clients (CRM conversations, lead fields, notes) reaches the agent as data. Combining CRM read access with “Write immediately” for money is risky — a client's message could push the agent into a payment; keep money on approval.

## License

MIT — see [LICENSE](LICENSE).

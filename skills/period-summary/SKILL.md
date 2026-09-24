---
name: period-summary
description: Money and activity summary for a period — revenue, other income, expenses, net by day/cashbox/category, cash balances, new bookings and clients in the period. Use when the user asks "how much did we earn", "revenue for August", "summary for last week", or "what went through the cashboxes".
---

# Period summary

1. Call `company_reference` first (branches, local today, currency).
2. Resolve the period to ISO dates in the branch's local calendar (e.g. "last week" → Monday..Sunday before today). Windows are capped at 366 days.
3. Money: `money_report(from, to, group_by: day | cashbox | category, operation: both | income | expense)`. The headline comes from `revenue`, `other_income`, `expenses` and `net` — each a map of currency to amount; there is no `totals` key. `rent_to_own` (when present) is money received under rent-to-own contracts: it mixes rent, interest, late fees and buyout, so report it as its own line and never add it to revenue or net. `turnover` holds deposits, internal transfers and client balance top-ups as two gross directions (`in` / `out`) — it is reference only and is NOT part of net. `groups` gives the breakdown, `balances_at_to` the cashbox balances at the end of the window (read `balances_note`).
4. Per-car economics is not in `money_report`: use `fleet_economics(from, to)` — it also gives utilisation, margin and payback. It is an owner-level tool and is absent for a rank-and-file employee.
5. Activity: `search_bookings(created_from, created_to)` → `total` new bookings; `search_clients(created_from, created_to)` → `total` new clients. For "who changed what" use `recent_changes` instead.

If `money_report` is not in your tool list, the key's role does not include finance data — say so and offer the activity part only. Answer in the user's language: headline (income, expense, net, currency), then the breakdown table, then cash balances, then activity counts. Sums are per currency without conversion. Give `ui_url` only for entities the user may want to open.

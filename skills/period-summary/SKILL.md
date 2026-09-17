---
name: period-summary
description: Money and activity summary for a period — income, expense, net by day/car/cashbox/category, cash balances, new bookings and clients in the period. Use when the user asks "how much did we earn", "revenue for August", "summary for last week", or "what went through the cashboxes".
---

# Period summary

1. Call `company_reference` first (branches, local today, currency).
2. Resolve the period to ISO dates in the branch's local calendar (e.g. "last week" → Monday..Sunday before today). Windows are capped at 366 days.
3. Money: `money_report(from, to, group_by: day | car | cashbox | category, operation: both | income | expense)`. Use `totals` per currency for the headline and `groups` for the breakdown; `balances_at_to` gives cashbox balances at the end of the window (read `balances_note`).
4. Activity: `search_bookings(created_from, created_to)` → `total` new bookings; `search_clients(created_from, created_to)` → `total` new clients. For "who changed what" use `recent_changes` instead.

If `money_report` is not in your tool list, the key's role does not include finance data — say so and offer the activity part only. Answer in the user's language: headline (income, expense, net, currency), then the breakdown table, then cash balances, then activity counts. Sums are per currency without conversion. Give `ui_url` only for entities the user may want to open.

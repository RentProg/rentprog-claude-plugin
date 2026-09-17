---
name: debtors
description: Who owes the company money and how much — open receivables per booking, overdue by N days, deposits held. Use when the user asks about debts, unpaid bookings, debtors, or who has not paid.
---

# Debtors

Debt in RentProg is computed per booking: `to_pay = total − paid`. The only source is the `receivables` tool. `search_clients(debtor: true)` is a manual flag in the client card, not a sum — use it only when the user asks "who is marked as a debtor".

1. Call `company_reference` first (branches, local today, currency).
2. `receivables(overdue_days_min: N)` — N from the question ("more than 5 days" → 5; default 0). Add `branch_id` when the user names a branch, `client_id` for one client (find the id with `search_clients(query)` first). For deposits held: `include_deposits: true`.
3. Use `totals` (whole selection, not the page) for the headline: count, `to_pay_sum`, `deposits_sum`, currency. Then list items sorted as returned (most overdue first): booking number, client `display_name`, car code, `to_pay`, `overdue_days`, `ui_url`.
4. If the user asks about one client's total, prefer `client_dossier(client_id)` → `money.debt_total`; for line items stay with `receivables(client_id)`.

Answer in the user's language. Do not translate stored values (booking states, categories) — show them as they are, with `label` when present. Never guess amounts; if `has_more` is true, say how many more rows exist and offer the next page.

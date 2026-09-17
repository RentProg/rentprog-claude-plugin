---
name: morning-brief
description: Morning briefing for a rental-fleet manager — today's handovers and returns, overdue returns, what changed overnight, cars free today with prices. Use when the user asks for a brief, a summary of the day, "what's on today", or a daily report.
---

# Morning brief

You have RentProg tools (MCP server `rentprog`). Every fact comes from a tool call — never from memory.

1. Call `company_reference` first. Take each branch's local `today`, the branch ids and the dictionaries.
2. Schedule: `schedule(from: today, to: today, type: both)`. If the user has one branch, pass no `branch_id`; for a group, run once per branch or once for all and group by `branch_id`.
3. Overdue returns: `schedule(from: today − 7 days, to: today, type: returns)` → items with `overdue: true`.
4. Overnight changes: `recent_changes(since: yesterday 18:00 local)` — list `kind`, entity label, `by.name`, and `changes` for state changes.
5. Free cars with price: `availability_quote(start_date: today, end_date: tomorrow)`. Mention `summary.without_price_count` if it is not zero.

Answer in the user's language, in this order: **Handovers today** (time, car, client, `money.to_pay` if > 0) · **Returns today** (time, car, client; mark `overdue`, `withheld` when a charge was recorded) · **Overdue returns** · **Overnight changes** · **Free cars today** (code, name, price_per_day, currency). Use a compact table per section. Give the `ui_url` for anything the user may want to open. Times are already in branch local time — do not convert them. If a tool returns an error with `allowed`, pick a value from `allowed` and retry once; otherwise report the error plainly.

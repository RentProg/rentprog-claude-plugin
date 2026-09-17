---
name: client-dossier
description: Dossier on one client before a call — rental history, open debt, deposits, fines, expiring documents, flags. Use when the user asks "tell me about client X", "is this client reliable", or "prepare me for a call".
---

# Client dossier

1. Call `company_reference` first.
2. Find the client: `search_clients(query: <name, phone or email>)`. If several match, show them and ask which one; otherwise take `items[0].id`.
3. `client_dossier(client_id)` — aggregates: history (bookings, completed, cancelled, first/last rental, favourite car classes), money (`debt_total`, `open_bookings_count`, `deposits_held`), fines (count, unpaid), documents expiring within 30 days, flags (debtor, problems).
4. Only if the user asks for stored card fields (phone, email, balance, full document dates): `get_card(kind: client, id)`. Only if line items are needed: `receivables(client_id)`, `fines(client_id, from, to)`, `search_bookings(client_id)`.

Answer in the user's language as a short profile: who (display_name, category label, source), history in one line, money in one line, fines, documents expiring (kind, date, days left), flags — then the `ui_url` of the card. If some fields are missing from the response, the key's role does not allow them; say so instead of guessing. Never invent contact details.

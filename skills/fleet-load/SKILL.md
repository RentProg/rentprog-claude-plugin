---
name: fleet-load
description: Fleet load right now — how many cars are rented out, free, in service; idle cars; cars with maintenance or insurance due. Use when the user asks about fleet utilisation, idle cars, what is in repair, or upcoming maintenance.
---

# Fleet load

Fleet state is a snapshot from `fleet_status`. It has no prices and no date windows: prices for dates are `availability_quote`, handovers/returns are `schedule`, catalog attributes are `search_cars`.

1. Call `company_reference` first (branches, `car_states` with labels, `maintenance_types`).
2. `fleet_status()` for the snapshot; add `branch_id` when asked about one branch, `include_inactive: true` when the user asks about inactive cars.
3. Idle cars: `fleet_status(idle_days_min: N)` (default 7 for "a week").
4. Maintenance: `fleet_status(maintenance_due_in_days: N, maintenance_type: technical | insurance | payments)` — take the type value from `company_reference.maintenance_types`.

Answer in the user's language: headline from `summary` (total_active, in_rent, free, inactive, maintenance_due / maintenance_overdue), then state counts using the `car_states` labels, then a table of rows: car code and name, state label, current booking end (`current_booking.end_at`), next booking, `idle_days`, maintenance items due with `due_date`/`due_mileage` and `overdue`. Give `ui_url` for cars the user may want to open. Do not infer utilisation percentages over a period — that is not available; say so if asked.

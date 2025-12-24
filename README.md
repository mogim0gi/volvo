# Volvo + Home Assistant (Nordpool Smart Charging Dashboard)

A Home Assistant configuration repo for a **Volvo EV/PHEV charging dashboard** with **Nordpool price visualization** and a **smart “dynamic limit”** concept (plus a clean Lovelace UI built with custom cards).

This project is focused on:
- displaying **electricity prices** (Nordpool),
- computing + showing a **dynamic charging limit**,
- presenting a **charging “clock” / schedule text** for the user,
- and controlling a **charger / breaker switch** from the UI.

---

## What’s inside

- **Lovelace UI** blocks:
  - compact KPI row(s) (price, ready-by, status + SoC)
  - charge mode selector (Auto / Manual / Force)
  - manual override slider (conditional)
  - Nordpool chart (ApexCharts)
  - status buttons (plug / charging state / breaker)

- **Template sensors** (typical patterns used in this setup):
  - combined Nordpool graph attributes (for ApexCharts `data_generator`)
  - EV smart charging “brain” sensor (limit + schedule_text + UI status/reason)
  - optional power/cost rollups

---

## Requirements

### Home Assistant
- Home Assistant Core / OS (recent versions recommended)

### Integrations / entities you’ll need
You will need entities similar to:
- `sensor.nordpool_*` (Nordpool integration)
  - with attributes like `raw_today`, `raw_tomorrow`, `current_price`
- Volvo integration sensors, e.g.:
  - `sensor.volvo_*_battery` (SoC %)
  - `sensor.volvo_*_charging_status`
  - `sensor.volvo_*_charging_connection_status`
- Charger control:
  - `switch.*` (your breaker/charger relay)

Inputs used by the UI logic typically include:
- `input_select.ev_charge_mode`
- `input_datetime.ev_ready_by`
- `input_number.ev_limit_override_value`

### Custom cards (frontend)
This dashboard commonly uses:
- `stack-in-card`
- `button-card`
- `apexcharts-card`
- `layout-card` + `grid-layout`
- `mushroom-number-card` (optional)
- `mini-graph-card` (optional)
- `card-mod` (optional but nice)

Install via HACS, then add to Lovelace resources if needed.

---

## Installation

1. **Copy YAML**
   - Put template sensors into your HA config (e.g. `templates/` or a package).
   - Put Lovelace card YAML into your dashboard (manual YAML mode or UI editor “Manual”).

2. **Adjust entity IDs**
   - Search/replace Volvo sensor names, Nordpool sensor name, and your breaker switch.

3. **Restart / reload**
   - Reload template entities (or restart HA) after adding template sensors.

4. **Verify**
   - Developer Tools → States:
     - confirm `raw_today/raw_tomorrow` exist
     - confirm your “combined graph data” sensor has `attributes.prices` populated
   - Developer Tools → Template:
     - verify schedule/limit templates render without errors

---

## Notes & design decisions

### 15-minute vs 1-hour performance
If your Nordpool integration exposes 15-minute slots, rendering a 36h span can get heavy.
A practical compromise:
- store raw points in attributes,
- but `group_by: 1h` in ApexCharts (fast + readable).

### Nordpool “duplicate slots”
Some Nordpool setups return repeated time entries (often due to how raw attributes are built or refreshed).
Mitigation pattern:
- dedupe by `start|end` key before selecting / rendering.

### Using `this` in templates
`this` works **inside template entities**, but not when you paste the same template into **Developer Tools → Template**.
For dev tools testing, replace:
- `this.attributes.x` with a direct `state_attr('sensor.your_sensor', 'x')`

---

## Troubleshooting

- **Chart empty**
  - Check the `prices` attribute exists and is valid JSON-like output (list of `[ms, value]` pairs).
  - Confirm timestamps are in **milliseconds** for ApexCharts.

- **UI freezes / becomes unresponsive**
  - Reduce points (use `group_by` 1h)
  - Avoid heavy JS in `data_generator`
  - Avoid multiple charts with long spans on the same view

- **Limit shows Max (999)**
  - If you use `999` as a sentinel, you can hide it in-chart using a transform:
    - `return x >= 999 ? null : x;`
  - Keep the header showing “Max” separately if you want.

---

## Customization ideas

- Add a “charging window” overlay (cheap hours) **only if** it stays performant.
- Add a second chart series for “selected charging hours” (binary 0/1) to visualize the clock.
- Add per-day min/avg/max stats to the header for quick decisions.

---

## Security / privacy

If you store this repo publicly:
- avoid committing tokens, credentials, VINs, or private endpoints
- prefer using `secrets.yaml` for sensitive values

---

## License

Choose one:
- MIT (simple, permissive)
- Apache-2.0 (explicit patent grant)
- “All rights reserved” (if you don’t want reuse)

---

## Screenshots

> Add screenshots/GIFs here once you’re happy with the final dashboard layout.

---

## Credits

Built with Home Assistant + HACS custom cards and Nordpool pricing data.

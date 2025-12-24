# Volvo + Home Assistant (Nordpool Smart Charging Dashboard)

A Home Assistant configuration repo for a **Volvo EV/PHEV charging dashboard** with **Nordpool price visualization** and a **smart “dynamic limit”** concept (plus a clean Lovelace UI built with custom cards).

<img width="460" height="554" alt="image" src="https://github.com/user-attachments/assets/46089f71-c415-4f81-8563-79034efde8fb" />



https://github.com/user-attachments/assets/03d9916b-e84e-435d-9114-4e3e8f54f9c6




This project is focused on:
- displaying **electricity prices** (Nordpool),
- computing + showing a **dynamic charging limit**,
- presenting a **charging “clock” / schedule text** for the user,
- and controlling a **charger / breaker switch** from the UI.

---
## Disclaimer - ** There will be no support provided, use at your own risk. **

This is a personal project that is built around:
- [official Volvo integration](https://www.home-assistant.io/integrations/volvo/)
- [Custom Nordpool price sensor](https://github.com/custom-components/nordpool)
- [Tuya integration for electricity data via breaker switch](https://www.home-assistant.io/integrations/tuya/)

<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/8288b64b-cf45-4a70-8646-5423bb7033df" />


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

Inputs used by the UI logic include:
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

### Handling Nordpool updates
Nordpool updates its tomorrow prices around 14.00 , to handle charge scenarios - a price from today's midnight to 7AM are used as placeholder value for tomorrow night prices until new prices are received.

### 95-100% SoC
My car takes full 35 minutes to charge and balance the cells, this is hardcoded into the charge logic.



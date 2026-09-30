# Home Dashboard

`home_dashboard.yaml` is a glass-panel Lovelace dashboard in the style of a
modern "home at a glance" wall tablet: header with greeting, people, weather
and status pills, then panels for Home Energy, Weather, Quick Actions, Rooms,
Calendar, Car and Music.

## Install

1. **HACS → Frontend**, install:
   - [button-card](https://github.com/custom-cards/button-card)
   - [layout-card](https://github.com/thomasloven/lovelace-layout-card)
   - [card-mod](https://github.com/thomasloven/lovelace-card-mod)
2. Add the `input_select.dashboard_room_tab` helper from `helpers.yaml`
   (or create a Dropdown helper with options `Living` / `Bedrooms & baths`).
3. Optional: add the **Time & Date** integration so `sensor.time` exists — it
   makes the header clock tick every minute.
4. **Settings → Dashboards → Add dashboard**, open it, **⋮ → Edit dashboard →
   ⋮ → Raw configuration editor**, paste the whole of `home_dashboard.yaml`,
   save.

Entities that don't exist yet don't break anything; the affected field shows `—`,
or the tile shows as off. Swap in your real IDs with find & replace.

## Entities used

Already in your setup (from `automations/weather_alerts.yaml`):

| Where | Entity |
| --- | --- |
| Header status pill (shows active NWS alert, red for tornado/hurricane) | `sensor.nws_alerts_32803` |
| Header lightning pill (amber under 11 mi) | `sensor.home_lightning_distance` |
| Music panel + Lanai "TV on" hint | `media_player.lanai_north` |

Placeholders — rename to match yours:

| Panel | Placeholder entity | Notes |
| --- | --- | --- |
| Header | `person.b_will`, `person.karla` | Avatars dim when not home |
| Header / Weather | `weather.home` | |
| Energy | `sensor.solar_power`, `sensor.grid_power`, `sensor.home_power` | Watts. Grid: **+ import / − export** |
| Energy | `sensor.battery_level`, `sensor.battery_power` | %, W. Battery: **+ charging / − discharging** |
| Energy | `sensor.solar_energy_today`, `sensor.home_energy_today`, `sensor.grid_import_today`, `sensor.grid_export_today` | kWh (utility_meter daily sensors work well) |
| Energy | `rate: 0.14`, `currency: $` | Grid cost per kWh for the "≈ $" line |
| Quick actions | `script.good_morning` | |
| Quick actions | `climate.downstairs`, `climate.upstairs` | "n of N on" |
| Quick actions | *(all `light.*`)* | Counts every light on, ignoring groups; tap turns all off (with confirmation) |
| Quick actions | `switch.water_heater`, `vacuum.robot_vacuum`, `switch.pool_pump`, `switch.sprinklers`, `cover.garage_door` | Vacuum: tap = start, hold = dock, double-tap = details |
| Rooms | `light.kitchen`, `light.living_room`, `light.dining_room`, `light.office`, `light.lanai`, `light.primary_bedroom`, `light.primary_bathroom`, `light.guest_bedroom`, `light.bedroom_2`, `light.hall_bathroom`, `light.laundry` | Light groups show "n lights". Also listed under `room_lights` and each tab's `lights` for the counters |
| Rooms | `sensor.<room>_temperature` | Optional per tile |
| Rooms | `media_player.living_room_tv` | Shows "TV on" when lights are off |
| Calendar | `calendar.family`, `calendar.b_will`, `calendar.karla`, `calendar.trash_recycling` | Shows each calendar's next event; "Trash day today" appears when an event titled trash/garbage/recycling/bin is today |
| Car | `sensor.car_battery_level`, `sensor.car_range`, `lock.car_doors`, `binary_sensor.car_charging`, `number.car_charge_limit`, `device_tracker.car` | Also feeds the car node in the energy flow |

## Tapping

- Energy panel → `/energy`. Calendar panel → `/calendar`.
- Room tiles: tap toggles, hold opens details.
- Garage and sprinklers ask for confirmation.

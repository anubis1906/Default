# Weather Alerts Automation

`weather_alerts.yaml` extends the original lightning-detector automation with a
second branch that fires a critical phone notification when the National
Weather Service issues a **Tornado Warning** for the 32803 zip code
(Orlando / Orange County, FL).

## Prerequisite: NWS alert sensor

The automation reads `sensor.nws_alerts_32803`, which is **not built in** to
Home Assistant — a source of active NWS alerts by zone has to be created:

1. Install the **NWS Alerts** custom integration via HACS
   (`mattdavis90/ha-nws-alerts`).
2. During setup, configure it with:
   - **Zone ID:** `FLZ344` (the NWS public forecast zone covering Orange
     County, FL / zip 32803)
   - **Name:** `32803` (so the entity becomes `sensor.nws_alerts_32803` —
     rename here or in the automation if you use a different name)
3. Confirm in **Developer Tools → States** that the sensor appears and, once
   an alert is active, has an `alerts` attribute listing the active NWS
   alerts (each includes an `event` field, e.g. `"Tornado Warning"`).

The automation's tornado condition checks for the literal text
`Tornado Warning` inside that `alerts` attribute, so it doesn't depend on the
exact attribute schema of whichever NWS alerts integration/fork you use —
only that the alert's event name shows up somewhere in it.

## How it works

- `trigger: lightning` — unchanged from the original automation.
- `trigger: tornado_warning` — fires on any state change of
  `sensor.nws_alerts_32803`; the `choose` block only runs the tornado
  actions when a Tornado Warning is actually present.
- Tornado phone notifications set `interruption-level: critical` (iOS) so
  they can break through Focus/Do Not Disturb — this requires the
  [critical alerts entitlement](https://companion.home-assistant.io/docs/notifications/critical-notifications/)
  to be enabled for the Home Assistant Companion app on each phone.

# Nest Smart Climate Automation

`nest_smart_climate.yaml` puts the Nest thermostat into Eco mode whenever the
house is empty, and otherwise keeps it on a target temperature that shifts
with time of day and current outdoor temperature/dew point.

## Prerequisite: entity IDs

This automation references a few entities that almost certainly don't match
your setup out of the box — rename them in the YAML (or rename your HA
entities to match) before enabling it:

| Entity in the automation | What it should be |
| --- | --- |
| `climate.nest_thermostat` | Your Nest thermostat entity from the [Google Nest (SDM) integration](https://www.home-assistant.io/integrations/nest/). Must support the `eco` preset mode. |
| `person.b_will`, `person.karla` | The `person.*` entities for you and your wife (Settings → People). |
| `sensor.outdoor_temperature` | An outdoor temperature sensor — e.g. from a WeatherFlow Tempest, Ecowitt station, or a `weather.*` entity's `temperature` attribute. |
| `sensor.outdoor_dew_point` | An outdoor dew point sensor. Tempest/Ecowitt stations report this directly; if yours doesn't, add a [template sensor](https://www.home-assistant.io/integrations/template/) that computes it from outdoor temperature + humidity. |
| `notify.mobile_app_b_wills_iphone`, `notify.mobile_app_karlas_iphone` | Same notify targets used in `weather_alerts.yaml`. |

## How it works

- **Away → Eco.** When both people are `not_home` for 10 minutes straight
  (the delay avoids flapping on a flaky GPS update), the thermostat is set
  to `preset_mode: eco` and both phones get a quiet notification.
- **Home → scheduled comfort temperature.** Whenever someone arrives home,
  every 30 minutes, or whenever the outdoor temperature/dew point sensors
  change, the automation (as long as at least one person is home) takes the
  thermostat out of Eco and computes a target temperature:
  - **Time-of-day base:** 74°F overnight (10pm–6am), 72°F early morning
    (6am–9am), 76°F during the day (9am–5pm), 75°F in the evening
    (5pm–10pm).
  - **Dew point adjustment:** −1°F if the dew point is above 68°F (muggy —
    cool a bit more for comfort), +1°F if it's below 55°F (dry — no need to
    overcool).
  - **Outdoor temperature adjustment:** +1°F if it's under 78°F outside
    (little cooling load, save energy), −1°F if it's over 92°F (extreme
    heat — get ahead of the AC's workload).
  - The result is clamped to a 68–78°F range before being sent to the
    thermostat.
- These numbers are a starting point — adjust the base temperatures and
  thresholds in the `variables` block to match your own comfort preferences.

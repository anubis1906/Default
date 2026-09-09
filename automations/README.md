# Pool Light Automations

The original single "Pool Light" automation turned the light on at sunset,
then used `wait_for_trigger` to sit idle until sunrise before turning it back
off. That meant one automation run spanned all night, which makes it harder
to reason about, edit, or manually re-trigger either half independently.

It's now split into two independent automations:

- `pool_light_on.yaml` — triggers at sunset and turns the pool light on.
- `pool_light_off.yaml` — triggers at sunrise and turns the pool light off.

Each keeps the original device/entity IDs and can be imported into Home
Assistant separately (Settings → Automations → Create Automation → Edit in
YAML, then paste).

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

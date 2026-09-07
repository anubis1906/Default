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

---

# Trash Day Reminder Automation

`trash_day_reminder.yaml` announces and pushes a reminder the night before
trash/recycling pickup, timed off a `calendar` trigger.

## Prerequisite: `calendar.garbage`

The automation triggers off `calendar.garbage`, which is **not built in** —
it has to be created by importing the municipal collection calendar feed:

```
webcal://api.recollect.net/api/places/C9691F10-E76C-11E8-B864-D2D0B33ECFC0/services/675/events.en-US.ics?client_id=518A7958-AAD2-11F1-9E3B-B6C0F8F98153
```

`webcal://` is just a hint to a calendar client to fetch over HTTPS — swap
the scheme before giving the URL to Home Assistant:

```
https://api.recollect.net/api/places/C9691F10-E76C-11E8-B864-D2D0B33ECFC0/services/675/events.en-US.ics?client_id=518A7958-AAD2-11F1-9E3B-B6C0F8F98153
```

Set it up with either:

1. **Core "Remote Calendar" integration** (Settings → Devices & Services →
   Add Integration → *Remote Calendar*, available since HA 2025.2). Paste
   the `https://` URL above and name the calendar **`Garbage`** so Home
   Assistant generates the entity ID `calendar.garbage` (rename the entity
   afterwards, or update this automation's `entity_id`, if you use a
   different name).
2. **`ics_calendar` custom integration** (HACS: `essandess/ha-ics-calendar`)
   if you want the source configured directly in YAML, e.g. in
   `configuration.yaml`:

   ```yaml
   calendar:
     - platform: ics_calendar
       calendars:
         - name: "Garbage"
           url: "https://api.recollect.net/api/places/C9691F10-E76C-11E8-B864-D2D0B33ECFC0/services/675/events.en-US.ics?client_id=518A7958-AAD2-11F1-9E3B-B6C0F8F98153"
   ```

Confirm in **Developer Tools → States** that `calendar.garbage` appears and,
once the integration polls the feed, shows the next pickup as its upcoming
event.

## How it works

- `trigger: calendar`, `event: start`, `offset: "-04:01:00"` — ReCollect
  publishes pickup days as all-day events (event "start" = midnight of
  collection day), so an offset of 4 hours and 1 minute *before* that
  fires the automation at **7:59 PM the evening before** pickup.
- Actions chime the kitchen speaker, announce "Tomorrow is trash day..." via
  TTS, and push a 🗑️ reminder notification to both phones.

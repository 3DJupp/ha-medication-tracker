# HA Medication Tracker

A Home Assistant medication reminder with actionable notifications (iOS and
Android), a rich status sensor, and a compact Mushroom badge for dashboards.

Based on and inspired by [Advanced medication reminder (Android)](https://github.com/Aohzan/hass-blueprints/blob/main/blueprints/medication_reminder_android.yaml)
by [Aohzan](https://github.com/Aohzan). This project extends the original idea
with an iOS/Android platform switch, a notification icon and channel,
snooze/timeout/repetition controls, and direct integration with helper entities
so a template sensor can distinguish more than on/off.

## What you get

The notification offers three responses: **Taken**, **Later**, **Skip**.
A template sensor derives one of six states from the helpers and the
reminder automation itself:

| State       | Meaning                                                        |
| ----------- | -------------------------------------------------------------- |
| `pending`   | Reminder has not fired yet today                               |
| `due`       | Reminder fired, no response yet (within a 2 h window)          |
| `postponed` | "Later" was pressed (holds for 45 min after the press)         |
| `taken`     | "Taken" was pressed, or the boolean was toggled manually       |
| `skipped`   | "Skip" was pressed — resolved, but consciously not taken       |
| `missed`    | No response and the 2 h window has passed                      |

Because the due/missed window is anchored to the automation's
`last_triggered`, the reminder time is configured **only in the blueprint**.
Changing it there moves everything else automatically.

## Repository layout

```
blueprints/medication_reminder.yaml   # the automation blueprint
packages/medication_tracker.yaml      # helpers + template sensor + manual-toggle automation
dashboard/badge.yaml                  # Mushroom template badge
```

## Installation

1. **Package**: enable packages in `configuration.yaml` if you have not:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

   Copy `packages/medication_tracker.yaml` into your `packages/` folder.

2. **Blueprint**: copy `blueprints/medication_reminder.yaml` to
   `blueprints/automation/<your_folder>/` and reload blueprints
   (Settings → Automations → Blueprints → ⋮ → Reload), or import it via the
   blueprint import dialog once this repository is public.

3. **Automation**: create an automation from the blueprint. Assign the three
   helpers from the package (`input_boolean.medication_tracker`,
   `input_datetime.medication_tracker_last`,
   `input_select.medication_tracker_action`), pick your device, platform and
   reminder time.

4. **Wire the sensor to your automation**: in the package, replace both
   occurrences of `automation.medication_reminder` with the entity id of the
   automation you just created.

5. Reload template entities and automations
   (Developer tools → YAML), or restart.

6. **Badge**: install [Mushroom](https://github.com/piitaya/lovelace-mushroom)
   via HACS, then add `dashboard/badge.yaml` as a badge to your view. Replace
   `YOUR_USER_ID` and the navigation path.

## The badge, and why it behaves the way it does

The badge is intentionally more than a state label. Use cases it covers:

**At-a-glance confirmation with timestamp.** Once resolved (`taken` or
`skipped`), the badge shows the response time (`08:12`) instead of a word.
"Did I take it this morning or am I remembering yesterday?" is answered
without tapping anything.

**Escalating urgency by color.** grey → orange → red mirrors
pending → due → missed. `postponed` gets yellow with a clock-alert icon, so a
consciously deferred dose looks different from silence: yellow means "I saw
it and decided later", orange/red means "nobody reacted".

**Skip is visible, not hidden.** `skipped` renders blue with `mdi:pill-off`.
It resolves the reminder loop but stays distinguishable from `taken` for the
rest of the day — useful when a partner or caregiver glances at the dashboard.

**Per-user visibility.** The `condition: user` block limits the badge to the
person it concerns. Medication status is personal; other household members on
the same dashboard never see it.

**Auto-hide when nothing needs attention.** The second visibility block shows
the badge only while the boolean is `off` (unresolved) *or* the last action
was `skipped`. After a normal "Taken" the badge disappears until the next
reminder cycle — the dashboard stays quiet when everything is fine, but a
skipped dose remains on display as a deliberate reminder. Remove this block
if you prefer a permanently visible badge (e.g. to keep the timestamp
around).

**Interactions.** Tap navigates to a detail view (put a `history-graph` or
`logbook` card on the status sensor there). Double-tap resolves the reminder
directly from the dashboard — the package's manual-toggle automation records
it as `taken` and stamps the time, so dashboard and notification stay
consistent.

## Notes

- All icons used are valid [Material Design Icons](https://pictogrammers.com/library/mdi/):
  `mdi:pill`, `mdi:pill-multiple`, `mdi:pill-off`, `mdi:clock-outline`,
  `mdi:clock-alert-outline`, `mdi:gesture-tap`.
- On iOS there is no per-notification icon; the Companion app always shows
  the app icon. The icon and channel inputs therefore only apply to Android.
  On iOS you can set the interruption level instead (`time-sensitive`
  recommended, `critical` requires a special entitlement).
- The template sensor is a trigger-based template. After editing the package,
  reload template entities — they do not pick up changes on their own.
- Multiple people: duplicate the package with different entity names, create
  one automation per person, and give each badge its own user id.

## Credits

- Original blueprint: [Aohzan/hass-blueprints](https://github.com/Aohzan/hass-blueprints) (medication_reminder_android.yaml)
- Badge built on [Mushroom](https://github.com/piitaya/lovelace-mushroom)

## License

MIT

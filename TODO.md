# Home Assistant TODO

## Open

### High Priority
- [ ] **Improve Home/Away Presence Logic** (automations.yaml:202-288)
  - Current logic only tracks Stephen's presence, doesn't include Lindsay
  - Should improve lighting automation tied to home/away status

### Medium Priority
- [ ] **Verify `sensor.rain_past_7_days_in` produces valid data** (sensors.yaml:10-16)
  - Statistics sensor depends on `sensor.rain_cumulative_in` (integration of OWM rain rate)
  - Entity was missing from `core.restore_state` — may be unavailable
  - Need to confirm OWM `current_rain` reports a rate (mm/hr) vs a volume (mm) for the integration sensor to work correctly
  - Irrigation system depends on this sensor being accurate

## Completed

### 2026-04-19
- [x] **Monthly Smoke Detector Test Reminder** (alerts.yaml, automations.yaml, templates.yaml, input_booleans.yaml, input_datetime.yaml)
  - Fires at 8 AM on the first Saturday of each month to both parents' phones
  - If ignored: nags again at ~6 PM, then daily at ~6 PM until confirmed
  - Actions: "Tested!" (dismisses), "Remind Me Tonight" (snoozes to 7 PM), "Remind Me Tomorrow" (snoozes to 8 AM next day)
  - Snooze tracked via `input_datetime.smoke_detector_snooze_until`; confirmation date tracked via `input_datetime.smoke_detector_last_tested`
  - Requires full HA restart to activate alert; template sensor and automations can hot-reload

### 2026-04-11
- [x] **Rewrite Family Room TV Input Manager** (automations.yaml)
  - Replaced "Set TV with Source" automation with generalized input-switching logic
  - New device turns on → steals focus (wakes TV if needed, retries source select up to 4x)
  - Device turns off → switches to remaining active device, or turns off TV if none active
  - Nintendo Switch guard: TV stays on if source is "Switch" (no HA entity to track)
  - Chromecast idle/ambient won't steal focus; only playing/buffering switches input
  - Xbox taking focus now pauses the Chromecast so it doesn't play in background
  - Added "LGTV Off - Stop Chromecast" automation to stop casting when TV is turned off
  - Removed disabled predecessor automation (id: 1764722098326)
  - All 11 test scenarios passed
  - Added "Pause Media on Input Switch" automation: pauses Chromecast and turns off Xbox when TV input switches away from them (covers both automation and manual remote switches)

### 2026-03-01
- [x] **Fix HVAC "Remind When Home" false trigger** (input_booleans.yaml, automations.yaml)
  - Tapping "Drain Bleached" was causing a false "Welcome home! Don't forget to bleach..." reminder because the condition checked `last_triggered` on the notification actions automation — which fires for any button press
  - Added `input_boolean.hvac_remind_when_home` flag that is only set when "Remind Me When Home Today" is tapped
  - Changed "HVAC Remind When Home Today" condition to check the boolean instead of `last_triggered`
  - Boolean is turned off immediately after the reminder fires

### 2026-02-27
- [x] **Fix Lawn Irrigation System** (templates.yaml, alerts.yaml, sensors.yaml)
  - Fixed entity name typo: `sensor.rain_in_past_7_days_in` → `sensor.rain_past_7_days_in` in `binary_sensor.weekly_rain_below_target` and alert message
  - Replaced placeholder notifier `mobile_app_your_phone` → `mobile_app_stephen_iphone`
  - Removed unused `binary_sensor.hot_day_24h_max_80f` template
  - System was completely non-functional due to these bugs; now operational pending rain sensor verification

- [x] **Fix all alert notifiers** (alerts.yaml)
  - Battery, hot water sensor, and HVAC bleach alerts all had `mobile_app_stephens_iphone` (typo, doesn't exist)
  - Corrected all three to `mobile_app_stephen_iphone`
  - This was likely why HVAC bleach reminders weren't showing up

- [x] **Bank of Dad transaction feedback** (scripts.yaml:422+)
  - Added zero-amount guard — script aborts silently if amount is $0
  - Added push notification to `notify.parents_phones` after every successful transaction
  - Wife's failed deposit was likely a websocket hiccup with no feedback

- [x] **Remove work phone references** (automations.yaml, notify.yaml)
  - Removed `mobile_app_stephens_work_iphone` from IP change notification and `stephens_devices` group
  - HA app deleted from work phone

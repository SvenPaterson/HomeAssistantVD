# Claude Code Protocol for Home Assistant Configuration

## Overview
This file provides guidance for Claude Code when working with this Home Assistant configuration.

## File Structure
- `/config/configuration.yaml` - Main configuration file (includes, integrations, timers)
- `/config/automations.yaml` - All automation definitions
- `/config/scenes.yaml` - Scene definitions
- `/config/scripts.yaml` - Script definitions
- `/config/sensors.yaml` - Integration & statistics sensor platforms
- `/config/templates.yaml` - Template sensors and binary sensors
- `/config/alerts.yaml` - Alert definitions (require HA restart, no hot reload)
- `/config/notify.yaml` - Notify groups
- `/config/input_booleans.yaml`, `input_numbers.yaml`, `input_datetime.yaml` - Input helpers
- `/config/TODO.md` - Task tracking (open items and completed work log)

## Session Startup Protocol
At the start of each new session, **automatically perform these steps**:

1. **Review `/config/TODO.md`** and note open items
2. **Generate and read the Watchman report**:
   ```bash
   curl -X POST -H "Authorization: Bearer $SUPERVISOR_TOKEN" -H "Content-Type: application/json" http://supervisor/core/api/services/watchman/report
   ```
   Then read `/config/watchman-report.txt` to identify any issues
3. **Present a summary** to the user showing:
   - Open TODO items
   - Key issues from Watchman report
   - Ask: "What would you like to tackle today?"

## Best Practices
1. Always validate YAML syntax before suggesting changes
2. Use existing patterns and conventions from the codebase
3. Test automations and scenes after modifications when possible
4. **MANDATORY: Keep `/config/TODO.md` up to date.** When you complete a task, move it to the Completed section with today's date and a summary of what was done. When new work is identified, add it to the Open section with the appropriate priority. This file is the single source of truth for project tracking.

## Key Conventions
- **Notify services**: Stephen's phone is `notify.mobile_app_stephen_iphone`, Lindsay's is `notify.mobile_app_iphone_2`, both parents is `notify.parents_phones`
- **Alert notifiers** use the service name without `notify.` prefix (e.g. `mobile_app_stephen_iphone`)
- **Alerts require a full HA restart** to reload — they do not support YAML hot reload
- **UI-created entities** (input_boolean, input_number, input_select, input_text created via UI) are stored in `.storage/core.entity_registry`, not in YAML files
- **Lovelace dashboards** are stored as JSON in `.storage/lovelace.lovelace` — use Python to modify cleanly

## Known Quirks
- **Mushroom grow light** (`switch.shellyplugus_083af20077a4_switch_0`): unplugged, not currently growing
- **Basement TV Chromecast** (`media_player.basement_tv`): unavailable when TV is off (USB-powered)
- **Amcrest camera**: YAML-configured (configuration.yaml:62-68), privacy mode switch exposed to HomeKit

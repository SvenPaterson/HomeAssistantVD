# Claude Code Protocol for Home Assistant Configuration

## Overview
This file provides guidance for Claude Code when working with this Home Assistant configuration.

## Important Notes
- Claude Code has built-in context tracking between sessions
- Document significant architectural decisions or non-obvious configuration choices below
- Reference specific files and line numbers when documenting important details

## File Structure Notes
- **Configuration Files**: This is a Home Assistant configuration directory
- **Main Files**:
  - `/config/configuration.yaml` - Main configuration file
  - `/config/automations.yaml` - All automation definitions
  - `/config/scenes.yaml` - Scene definitions
  - `/config/scripts.yaml` - Script definitions
  - `/config/sensors.yaml` - Sensor templates
  - `/config/templates.yaml` - Template definitions

## Best Practices
1. Always validate YAML syntax before suggesting changes
2. Use existing patterns and conventions from the codebase
3. Test automations and scenes after modifications when possible

## Session Startup Protocol
At the start of each new session, **automatically perform these steps**:

1. **Review the TODO list** below and note pending items
2. **Generate and read the Watchman report**:
   ```bash
   curl -X POST -H "Authorization: Bearer $SUPERVISOR_TOKEN" -H "Content-Type: application/json" http://supervisor/core/api/services/watchman/report
   ```
   Then read `/config/watchman-report.txt` to identify any issues
3. **Present a summary** to the user showing:
   - Pending TODO items from the list below
   - Key issues from Watchman report
   - Ask: "What would you like to tackle today?"

**Note**: Watchman tracks missing/unavailable entities referenced in your configuration files. Regular reviews help maintain config health.

## Recent Work & Known Issues
- **2025-11-10**: HVAC Bleach Notification System Fixed
  - **Problem**: When accepting the monthly HVAC bleach reminder notification (or "remind me when home" notification), the backend updated correctly but notifications persisted in iPhone notification center
  - **Root Cause**:
    - Alert and automation notifications were missing notification `tag` parameter
    - Without a tag, iOS notifications can't be replaced or cleared programmatically
    - Each notification was independent, so clearing the alert state didn't remove old notifications
  - **Changes Made**:
    - Added `tag: "hvac-bleach-reminder"` to main alert (alerts.yaml:116)
    - Changed alert `done_message` to `clear_notification` (alerts.yaml:104)
    - Added `group: "hvac-maintenance"` for better organization (alerts.yaml:117)
    - Updated "HVAC Notification Actions" automation to explicitly clear notification when marking complete (automations.yaml:1232-1236)
    - Added tag to "HVAC Remind When Home Today" notification (automations.yaml:1283-1284)
  - **Impact**: Notifications now properly clear from iPhone when task is completed or alert resolves
  - **Status**: ✅ COMPLETE - All HVAC reminder notifications now share the same tag and clear properly

- **2025-11-03**: Battery Alert System Improvements
  - **Problem**: Battery alert didn't fire when master bedroom Govee sensor died
  - **Changes Made**:
    - Updated battery threshold from <15% to <30% (templates.yaml:54)
    - Changed alert from hourly to escalating schedule: 1hr → 6hrs → daily (alerts.yaml:6-9)
    - Improved notification format with iOS SF Symbols icons (alerts.yaml:14-26)
    - Added "Batteries Changed" action button (alerts.yaml:19-22)
    - Created automation to handle battery change notifications (automations.yaml:1388-1401)
  - **Impact**: Earlier warnings with better notification reliability

- **2025-11-03**: Amcrest Camera Privacy Mode Fix (IN PROGRESS)
  - **Problem**: Privacy mode button not working; `switch.amcrest_camera_privacy_mode` entity missing from HA and HomeKit
  - **Root Cause**:
    - YAML configuration was valid, but camera integration had connectivity issues
    - Camera API is working (verified via curl with digest auth)
    - Privacy mode (LeLensMask) confirmed supported by camera
  - **Investigation Findings**:
    - No UI integration available for Amcrest (YAML is correct method per docs)
    - Camera was experiencing HTTP 500 errors and RTSP connection failures
    - Last successful HA restart: Oct 26, 2025
  - **Fix Applied**:
    - Re-enabled YAML amcrest config (configuration.yaml:60-66)
    - Added explicit `name: Amcrest Camera` for consistent entity IDs (per docs recommendation)
    - HomeKit filter already configured correctly (configuration.yaml:36)
  - **Configuration**:
    ```yaml
    amcrest:
      - host: 192.168.1.72
        username: admin
        password: M4rbj9fHhd
        name: Amcrest Camera
        switches:
          - privacy_mode

    homekit:
      filter:
        include_entities:
          - switch.amcrest_camera_privacy_mode  # Auto-exposes to Apple HomeKit
    ```
  - **Next Steps** (USER ACTION REQUIRED):
    1. **Restart Home Assistant** (Settings > System > Restart)
    2. Verify `switch.amcrest_camera_privacy_mode` entity appears in HA
    3. Check HomeKit app for privacy mode switch (may need to reload HomeKit integration)
    4. Test automation at automations.yaml:977-997 (10min auto-enable timer)
  - **Status**: Configuration ready, awaiting HA restart to create switch entity

- **2025-10-23**: Watchman cleanup
  - **ESP32-S3-Korvo**: Removed from configuration (not in use anymore)
  - **kitchen_speaker**: Removed from `groups.yaml` (replaced with Home Assistant Voice PE)
  - **Mushroom grow light**: `switch.shellyplugus_083af20077a4_switch_0` is unplugged (not currently growing). Friendly name can be set via UI when needed.
  - **Basement TV Chromecast** (`media_player.basement_tv`): Shows as unavailable because TV USB port powers down when TV is off, which shuts down the Chromecast. Needs separate power supply to fix. Currently referenced in `sceness_BU.yaml` (backup file).

- **2025-10-23**: Scene entity cleanup - `scene.return_to_normal_volume` vs `scene.return_to_normal_volume_2`
  - **Root cause identified**: Two entities in entity registry with different unique IDs
    - `scene.return_to_normal_volume` - unique_id: `1726144246563` (ORPHANED - no YAML backing)
    - `scene.return_to_normal_volume_2` - unique_id: `1726121346563` (CORRECT - backed by `/config/scenes.yaml:34`)
  - **Fix applied**: Updated automation in `/config/automations.yaml:796` to reference `scene.return_to_normal_volume`
  - **Current status**: User deleted orphaned entity via UI, testing after restart
  - **Note**: The `_2` scene is actually the legitimate one backed by YAML. After orphan removal, may need to rename or reassign unique ID

- **2025-10-23**: Marantz/Living Room Hifi cleanup
  - **Removed**: `OFFICE: AC & HT` automation (automations.yaml) - Marantz no longer used as home theater receiver
  - **Updated**: `llm_script_for_music_assistant_voice_requests` (scripts.yaml:69) - Changed default player from `media_player.marantz_sr5011_2` to `media_player.living_room_hifi_airplay`
  - **Status**: Marantz now used as living room hifi via Music Assistant AirPlay casting

- **2025-10-23**: Home Status automation cleanup
  - **Removed**: `switch.all_lights` reference from "Home Status Actions" automation (automations.yaml:246)
  - **Current behavior**: Only sets thermostats to away/home mode based on presence
  - **See TODO**: Improve home/away logic and lighting automation

## TODO List
**Last updated**: 2025-11-03

### High Priority
1. **Improve Home/Away Presence Logic** (automations.yaml:202-288)
   - Current: Only tracks `person.stephengarden` location
   - Issues:
     - Doesn't account for other household members
     - `switch.all_lights` removed but no replacement for turning off lights when away
   - Desired improvements:
     - Consider all household members before switching to "Away"
     - Create automation to turn off indoor lights when alarm is set to away mode
     - Optional: Random light schedule to mimic occupancy during vacation

2. **Fix Lawn Watering System** (templates.yaml:84)
   - Dashboard and automations have never worked properly
   - Missing sensor: `sensor.rain_in_past_7_days_in` (templates.yaml:84)
   - Template "Weekly Rain Below Target" depends on this sensor
   - Needs investigation and repair

3. **Fix HVAC Bleached Notification System**
   - Entity: `input_button.hvac_bleached_today` (configuration.yaml)
   - Current: Monthly reminder works, but notification/alert actions don't work perfectly
   - Automation: "HVAC Notification Actions" (automations.yaml:1195+)
   - Needs debugging and refinement

### Medium Priority
4. **Music Assistant Entity Cleanup**
   - Two duplicate Music Assistant integrations detected (both at 192.168.1.100:8095)
   - Need to delete duplicate integration via UI
   - Consider naming convention for MASS entities (e.g., `_mass` suffix, "MA" prefix in friendly names)

### Low Priority / Optional
5. **Delete or move ESP32-S3-Korvo archive files**
   - Files in `esphome/archive/` causing Watchman warnings
   - Consider moving outside config directory or adding to Watchman ignore list

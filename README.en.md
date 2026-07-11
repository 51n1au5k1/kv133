# Home Assistant Configuration: Apartment 133

[Русская версия](README.md)

This repository contains a Home Assistant YAML configuration for lighting,
climate, security, household workflows, infrastructure monitoring, and
notifications. Most logic is organized with
[`homeassistant.packages`](https://www.home-assistant.io/docs/configuration/packages/),
so each package owns its related helpers, templates, timers, scripts, and
automations.

## Design principles

- Hysteresis for climate equipment and ventilation.
- State confirmation with `for:`, `delay_on`, `delay_off`, and `timer`.
- Minimum run/idle times to prevent rapid switching.
- Reading stabilization with filter, statistics, and template sensors.
- Startup guards and periodic watchdog reconciliation after HA restarts.
- Entity availability checks and conservative defaults.
- Runtime thresholds exposed as `input_number` helpers.

## Features

### Presence and system modes

- Andrei and Hanna presence from `person.*` with Wi-Fi fallback.
- Composite `resident_is_home` and `someone_is_home` sensors.
- Guest, silent, bathroom cleaning, and forced camera-off modes.
- Camera control based on real resident presence and guest mode.
- Lights, TVs, and the garment steamer are turned off while the home is empty.

### Lighting

- Bathroom day/night scenes, motion and door activation, shutdown timer, and
  false-trigger protection after startup and arrival.
- Power supervision and automatic recovery for Philips Wi-Fi spotlights.
- Hallway door, motion, illuminance, arrival-window, and doorbell workflows.
- Physical MQTT switch controls for the kitchen, living room, and office.
- Living-room cabinet light brightness follows illuminance; color temperature
  follows solar elevation.

### Climate and ventilation

- Averaged living-room temperature and humidity.
- Humidifier with 47/50% RH hysteresis, minimum run/idle times, and an
  open-balcony-door interlock.
- Air purifier with a five-minute PM2.5 mean, 20/10 µg/m³ hysteresis, and
  minimum run/idle times.
- Air-conditioner winter lockout, restart state handling, and balcony-door
  protection.
- Weekday away cooling that only turns off an AC session started by the same
  automation.
- Kitchen exhaust based on kitchen-to-living-room temperature and humidity
  deltas: two-minute start confirmation, 15-minute minimum run time,
  five-minute normalization confirmation, and ten-minute final ventilation.
  Renewed demand cancels pending shutdown.
- Bathroom humidity exhaust with hysteresis and a final cooldown period.
- Server-room fan control with temperature hysteresis and two-stage delayed
  shutdown.

### Washing machine

- A cycle starts after power remains above 8 W for one minute.
- Completion requires 20 continuous minutes at or below 2 W.
- Default minimum cycle duration is 30 minutes.
- Program pauses do not close the cycle or clear tracking flags.
- A watchdog restores tracking after restarts or missed state events.
- Telegram and Yandex Station completion alerts, a delayed laundry reminder,
  and socket power-loss/restoration alerts.

### Plants

- Ficus lyrata: soil moisture and temperature monitoring.
- Euphorbia leuconeura: soil moisture, temperature, and illuminance monitoring.
- One-hour smoothing, notification cooldown, recent notification history, and
  a pending-condition check when the resident returns home.
- Telegram notifications use local images from
  `/config/www/images/plants_alerts/`.

### Notifications and remote control

- Daily Telegram forecast at 07:45. Official Yandex Pogoda is primary;
  Open-Meteo is the fallback provider.
- Telegram alerts for doors, HA lifecycle, updates, Proxmox, ZFS, S.M.A.R.T.,
  critical services, and safe microclimate.
- HTML/inline-keyboard Telegram bot for rooms, lights, sockets, climate,
  system status, Proxmox control, and confirmed HA restart.
- Yandex Station TTS for doorbell, climate, CO₂, washing-machine, and system
  events. Silent mode suppresses voice output.

### Infrastructure and history

- MariaDB recorder with daily manual purge and a 90-day retention policy.
- Normalized CPU usage and host contribution for Proxmox VMs and LXCs.
- ZFS, S.M.A.R.T., updates, and critical add-on monitoring.
- Safe dew point, absolute humidity, and mold-risk calculation.

## Repository structure

| Path | Purpose |
| --- | --- |
| `configuration.yaml` | Entry point, packages, themes, recorder, logger, HTTP proxy |
| `includes/recorder.yaml` | MariaDB and history include/exclude policy |
| `packages/helper_main.yaml` | Presence, modes, cameras, and global helpers |
| `packages/helper_climate_common.yaml` | Shared climate infrastructure and balcony-door protection |
| `packages/helper_proxmox.yaml` | VM/LXC CPU normalization |
| `packages/room_*.yaml` | Room lighting, climate, and workflows |
| `packages/livingroom_summer_cooling.yaml` | Away-mode summer cooling |
| `packages/washing_machine.yaml` | Laundry cycle tracking and alerts |
| `packages/plants_common.yaml` | Shared plant scripts and notification cooldown |
| `packages/plant_*.yaml` | Per-plant monitoring rules |
| `packages/telegram_bot.yaml` | Telegram bot menus and commands |
| `packages/telegram_notifications.yaml` | Telegram alerts and weather forecast |
| `packages/telegram_helper.yaml` | Legacy Telegram notes; integration is configured in the UI |
| `packages/yandex_helper.yaml` | Yandex Station TTS scripts |
| `packages/yandex_notifications.yaml` | Yandex Station events and voice alerts |
| `packages/weather_variables.yaml` | Open-Meteo Russian condition and wind helpers |
| `secrets.yaml.sample` | Required secret names without real values |

## External dependencies

Integrations and devices are configured separately in Home Assistant. The
repository intentionally excludes `.storage`, API keys, and real secrets.

- MQTT/Zigbee events from physical controls and sensors.
- Xiaomi Home and Xiaomi Miio Philips Light.
- Yandex Station and official
  [Yandex Pogoda](https://github.com/yandex/pogoda-home-assistant).
- Built-in Open-Meteo integration as the forecast fallback.
- Telegram Bot, Proxmox VE, MariaDB, and Speedtest/Cloudflare sensors.
- HACS frontend resources `ha-yandex-icons` and `hass-hue-icons`.

Entity IDs, device IDs, IP addresses, and tokens are specific to the current
installation and must be replaced when reusing this configuration elsewhere.

## Deployment and updates

1. Create `secrets.yaml` using `secrets.yaml.sample` as a reference.
2. Configure UI integrations and verify all referenced entity IDs.
3. Pull changes in the Home Assistant `/config` repository:

   ```bash
   git pull
   ```

4. Validate before restarting:

   ```bash
   ha core check
   ```

5. Restart Home Assistant only after a successful check.

Tune operational thresholds through the corresponding `input_number` helpers.
After changing automation logic, inspect automation traces and source-sensor
graphs in History.

## Security

- Never commit `secrets.yaml`, tokens, or passwords.
- Commit or stash local Home Assistant edits before `git pull` to avoid merge
  conflicts.
- Telegram restart requires confirmation, but bot access must still be limited
  to approved chat IDs.


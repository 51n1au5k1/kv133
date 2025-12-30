# Home Assistant Configuration

## Introduction

This repository contains my Home Assistant configuration, designed to automate smart home management. It covers a wide range of functionality, including lighting automation, climate control, plant care, and notifications.

## Features

- **Lighting Automation**: Smart lighting control in various rooms ensures comfort and energy efficiency. Scenarios for the bathroom and home office allow lighting to be adjusted for specific tasks or times of day, increasing convenience and reducing energy consumption.

- **Climate Control**: Maintaining comfortable conditions: air conditioning, heating, humidification by hysteresis (on < low threshold / off > high), air purification by PM2.5 (hysteresis + anti-chatter + minimum operating/idle time), and kitchen exhaust fan by comparative temperature/humidity deltas with the living room (hysteresis, anti-spam, and delayed shutdown).

- **Plant Care**: Configurations for monitoring plant conditions, such as Euphorbia and Ficus lyrata, include notification mechanisms for watering, fertilizing, or changing environmental conditions. This ensures their optimal growth and health.

- **Notifications**: Integration with Telegram and Yandex allows for instant notifications about smart home events, whether it's security, system changes, or plant care reminders. This increases awareness and allows for timely responses to various situations.

## Key Principles (Technical)

- **Hysteresis** wherever possible (humidifier, purifier, kitchen exhaust fan) — no "twitching."

- **Anti-chatter/anti-spam** via `for:` and timers/flags (minimum operating/idle intervals).

- **Averaging readings** (template and statistics sensors) to reduce the impact of short-term spikes.

- **Safe defaults and protection against incorrect thresholds** (e.g., if low ≥ high — automatically corrected).

- **Fault tolerance on restart** — "min. time" checks account for HA restarts.

## Configuration Files

- `configuration.yaml` — the main configuration file, linking all components together.
- `helper_main.yaml`,`helper_mikrotik.yaml`,`helper_proxmox.yaml`, `variables.yaml` — helper functions and entities for smart home monitoring and control.
- `notifications_telegram.yaml` and `notifications_yandex.yaml`  — notification settings via Telegram and Yandex.
- Telegram (UI integration):
    - `telegram_helper.yaml` — starting with version 2026.5.0, configuration via UI, old service configurations: Telegram, `notify.*`, recipient groups.
    - `telegram_notifications.yaml` — notifications via Telegram.    
    - `telegram_bot.yaml` — bot scripts (commands, inline buttons, menus, device control).
- Plant monitoring:
    - `plant_common.yaml` — configuration for plant monitoring.  
    - `plant_euphorbia_leuconeura.yaml`, `plant_ficus_lyrata.yaml` — plant monitoring configuration.
- Rooms and domain packages
    - `room_bathroom.yaml`, `room_hallway.yaml`, `room_livingroom.yaml`, `room_workroom.yaml`, `room_bedroom.yaml`, `room_kitchen.yaml`, `room_server.yaml`: Automations for various rooms.
    - `room_bathroom_light.yaml` — Philips Zhirui (philips.light.downlight) lights via Xiaomi Philips Lights.
    - *New:* `helper_climate_common.yaml` — common climate automation mechanisms used in room packages (`room_livingroom_*.yaml`).
    - *New:* `room_livingroom_humidifier.yaml` — auto-mode humidifier by hysteresis (humidity < low / > high thresholds, anti-chatter, min. time).
    - *New:* `room_livingroom_air_purifier.yaml` — auto-mode purifier by PM2.5 (statistics smoothing, hysteresis, anti-chatter, min. time).
    - *Updated:* `room_kitchen.yaml` — exhaust fan by comparative T/RH deltas of kitchen to living room, transition to averaged living room sensors, delayed shutdown, anti-spam.

**Additionally**: Features of the implemented Telegram bot for smart home control
A multifunctional Telegram bot is implemented in the configuration, providing device control and real-time status updates for smart home systems. The bot uses interactive buttons and supports commands for convenient user interaction.

## Main bot functions:
📋 Main menu – select a room or system for control.
🔄 Status monitoring – display temperature, humidity, CO₂, PM2.5, door status, and other parameters in real time.
💡 Lighting control – turn lights on/off in the living room, bedroom, bathroom, home office, and kitchen.
🔌 Socket control – ability to turn sockets on and off for various devices (e.g., router, mini PC, humidifier, etc.).
🔇 Silent mode – enable/disable `input_boolean.silent_mode`, which mutes sound notifications during certain hours.
🌡 Climate control – receive current temperature and humidity data in different areas of the house.
🌍 Internet speed check – display current ping, download, and upload values via `sensor.speedtest_*`.
⚠️ Forced system restart – allows safely restarting Home Assistant from Telegram.

## Bot menu structure:
Each room or system is represented by buttons that lead to the corresponding section:

- 🏠 Living Room (/livingroom_control)
- 🍽 Kitchen (/kitchen_control)
- 🏢 Home Office (/workroom_control)
- 🛏 Bedroom (/bedroom_control)
- 🛁 Bathroom (/bathroom_control)
- 🚪 Hallway (/hallway_control)
- 🖥 System (/system_control)

**Functions:** statuses (T/RH/CO₂/PM2.5/doors, etc.), light/socket/ventilation control, internet speed, HA restart.
**UX:** HTML messages, inline buttons, automatic deletion of old messages, minimization of duplication via `script.*`.

*Configuration note: the bot itself is configured via UI integration; `allowed_chat_ids` and keys are stored in `secrets.yaml`; handlers are via `telegram_command`/`telegram_callback` events.*

## Telegram bot implementation features:
✅ Interactive interface – all commands and statuses are formatted in HTML, and button states are updated automatically.
✅ Deleting old messages – the bot clears previous commands to avoid cluttering the chat.
✅ Optimization via scripts – control is performed via `script.*`, minimizing code duplication in automations.
✅ Flexible control – each command automatically updates the current state of the room or system.

📌 Bot configuration file: `telegram_bot.yaml`
📌 Notifications file: `telegram_notifications.yaml`

⚙️ The bot provides a simple and convenient way to control your smart home without needing to access the Home Assistant web interface! 🚀

## Nota Bene

In "Silent" mode, Yandex.Station sound notifications are limited; humidification/purification logic can operate 24/7 (quiet devices).
Climate logic considers balcony door openings: devices are temporarily turned off, and upon closing, only those that were active are restored.
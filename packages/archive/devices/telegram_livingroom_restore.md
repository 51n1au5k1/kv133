# Возврат устройств в меню гостиной Telegram

Фрагменты нужны только после физического возврата устройства и активации его
пакета. Не добавляйте их заранее: прямые ссылки снова попадут в Watchman.

## Увлажнитель

В сообщение `script.send_livingroom_status` можно вернуть строку:

```jinja2
💧 <b>Увлажнитель:</b> {{ states('humidifier.xiaomi_p3_946a_humidifier') }}
```

В климатический ряд `inline_keyboard` добавить:

```jinja2
["🔴 Увлажнитель" if is_state("humidifier.xiaomi_p3_946a_humidifier", "off") else "🟢 Увлажнитель", "/toggle_livingroom_humidifier"]
```

Callback уже находится внутри архивного device-пакета.

## Очиститель

В сообщение вернуть строку:

```jinja2
💨 <b>PM2.5:</b> {{ states('sensor.zhimi_mb3_a5d3_pm25') }} µg/m³
```

В климатический ряд добавить:

```jinja2
["🔴 Очиститель" if is_state("fan.zhimi_mb3_a5d3_air_purifier", "off") else "🟢 Очиститель", "/toggle_livingroom_air_purifier"]
```

Callback уже находится внутри архивного device-пакета.

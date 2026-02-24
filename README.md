# Weather Dashboard

ESPHome configuration for the reTerminal E1002 displaying a weather dashboard with indoor conditions, current outdoor conditions, and a 3-day forecast.

## Display Layout

```
┌──────────────────────────────────────────────────────────────────────────┐
│  HEADER BAR  (y 0–58)  Navy bg / White text  │  Date + battery          │
├────────────────────────────┬─────────────────────────────────────────────┤
│  INDOOR  (x 0–380)         │  OUTDOOR  (x 400–800)                       │
│  Large °F   Humidity %     │  Large °F   Humidity %   Condition          │
│  (y 70–285)                │  (y 70–285)                                 │
├────────────────────────────┴─────────────────────────────────────────────┤
│  3-DAY FORECAST (y 295–480) — three equal panels of ~266 px each        │
│  Panel 0 (Today)  │  Panel 1 (Tomorrow)  │  Panel 2 (Day after)         │
└───────────────────┴──────────────────────┴──────────────────────────────┘
```

## Color Legend

| Color | Usage |
|-------|-------|
| Navy | Header background, day name labels |
| White | Header text |
| Blue | "INDOOR" label, low temperatures |
| Green | "OUTDOOR" label |
| Yellow | Sunny / partly cloudy conditions |
| Red | High temperatures, storm conditions |
| Black | Body text, rules |

## Prerequisites

### Home Assistant Integrations

- [OpenWeatherMap](https://www.home-assistant.io/integrations/openweathermap/) — provides `weather.openweathermap`
- Set your unit system to **Imperial** in HA (Settings → System → General) so temperatures arrive in °F

### Template Sensors (add to `configuration.yaml`)

The 3-day forecast requires individual template sensors because ESPHome can only read simple entity states, not forecast list attributes directly.

Add the following to your Home Assistant `configuration.yaml`:

```yaml
template:
  - sensor:
      - name: "Forecast Day0 Name"
        unique_id: forecast_day0_name
        state: >
          {{ as_timestamp(now()) | timestamp_custom('%A') }}

      - name: "Forecast Day1 Name"
        unique_id: forecast_day1_name
        state: >
          {{ (as_timestamp(now()) + 86400) | timestamp_custom('%A') }}

      - name: "Forecast Day2 Name"
        unique_id: forecast_day2_name
        state: >
          {{ (as_timestamp(now()) + 172800) | timestamp_custom('%A') }}

      - name: "Forecast Day0 Condition"
        unique_id: forecast_day0_condition
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[0]['condition'] }}

      - name: "Forecast Day1 Condition"
        unique_id: forecast_day1_condition
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[1]['condition'] }}

      - name: "Forecast Day2 Condition"
        unique_id: forecast_day2_condition
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[2]['condition'] }}

      - name: "Forecast Day0 High"
        unique_id: forecast_day0_high
        unit_of_measurement: "°F"
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[0]['temperature'] }}

      - name: "Forecast Day0 Low"
        unique_id: forecast_day0_low
        unit_of_measurement: "°F"
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[0]['templow'] }}

      - name: "Forecast Day1 High"
        unique_id: forecast_day1_high
        unit_of_measurement: "°F"
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[1]['temperature'] }}

      - name: "Forecast Day1 Low"
        unique_id: forecast_day1_low
        unit_of_measurement: "°F"
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[1]['templow'] }}

      - name: "Forecast Day2 High"
        unique_id: forecast_day2_high
        unit_of_measurement: "°F"
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[2]['temperature'] }}

      - name: "Forecast Day2 Low"
        unique_id: forecast_day2_low
        unit_of_measurement: "°F"
        state: >
          {{ state_attr('weather.openweathermap', 'forecast')[2]['templow'] }}
```

After adding these, restart Home Assistant and verify the sensors appear in Developer Tools → States.

## Secrets Required

The following keys must be present in your `secrets.yaml`:

```yaml
wifi_ssid: "your_network"
wifi_password: "your_password"
ap_password: "your_fallback_ap_password"
api_key: "your_base64_api_key="
ota_password: "your_ota_password"
```

## Flashing

```bash
esphome run weather-dashboard/weather-dashboard.yaml
```

## Notes

- The display refreshes every **30 minutes** (driven by the SHT40 `on_value` callback at `update_interval: 1800s`). Color ePaper takes ~30 seconds to complete a full refresh.
- `update_interval: never` on the display component is intentional — the SHT40 sensor triggers the update.
- Requires **ESPHome ≥ 2025.11.0** for the `Seeed-reTerminal-E1002` display model.
- The Arduino framework is mandatory for 8 MB PSRAM support.

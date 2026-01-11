# Neevo Propane Tank Monitor for Home Assistant

Monitor your Neevo propane tank levels directly in Home Assistant using the official Neevo API.

## Features

- **Real-time tank level monitoring** - Track your propane tank fill percentage
- **Automatic daily updates** - Sensor updates every 24 hours
- **Historical data** - Access last reading date and tank capacity information
- **Low maintenance** - Set it and forget it configuration

## Prerequisites

- Home Assistant installed and running
- Neevo account with active propane tank monitoring service
- Neevo username and password (requires setup in Neevo phone app and scanning of QR code on reader to register as home owner)

## Installation

### Step 1: Add Credentials to Secrets

Edit your `secrets.yaml` file (located in your Home Assistant config directory) and add your Neevo credentials:

```yaml
neevo_username: your_neevo_username
neevo_password: your_neevo_password
```

> **Note:** Replace `your_neevo_username` and `your_neevo_password` with your actual Neevo account credentials.

### Step 2: Add REST Sensor Configuration

Add the following configuration to your `configuration.yaml` file:

```yaml
rest:
  # -----------------------
  # Neevo Tank Sensor
  # -----------------------
  - resource: https://ws.otodatanetwork.com/neevoapp/v1/DataService.svc/GetAllDisplayPropaneDevices
    method: GET
    scan_interval: 1440
    headers:
      Content-Type: application/json
      User-Agent: HomeAssistant
    authentication: basic
    username: !secret neevo_username
    password: !secret neevo_password
    sensor:
      - name: neevotank_raw
        unique_id: neevotank_raw_sensor
        value_template: "{{ value_json[0].Level }}"
        json_attributes_path: "$[0]"
        json_attributes:
          - Level
          - LastReadingDate
          - TankCapacity
```

> **Note:** If you already have a `rest:` section in your configuration.yaml, add the new sensor under the existing `rest:` key rather than creating a duplicate.

### Step 3: Restart Home Assistant

After making these changes:

1. Check your configuration is valid: **Settings → System → Repair → Check Configuration**
2. Restart Home Assistant: **Settings → System → Restart**

## Usage

### Accessing the Sensor

Once configured, the sensor will be available as:
- **Entity ID:** `sensor.neevotank_raw`
- **State:** Current tank level percentage (0-100)

### Available Attributes

The sensor provides the following attributes:

- **Level** - Current tank fill percentage
- **LastReadingDate** - Timestamp of the last sensor reading
- **TankCapacity** - Total capacity of your propane tank

### Example Dashboard Card

Add a gauge card to your dashboard to visualize the tank level:

```yaml
type: gauge
entity: sensor.neevotank_raw
name: Propane Tank
min: 0
max: 100
severity:
  green: 50
  yellow: 25
  red: 0
```

### Creating Template Sensors

You can create additional template sensors for better formatting:

```yaml
template:
  - sensor:
      - name: "Propane Tank Level"
        unique_id: propane_tank_level
        state: "{{ states('sensor.neevotank_raw') }}"
        unit_of_measurement: "%"
        device_class: battery
        
      - name: "Propane Tank Capacity"
        unique_id: propane_tank_capacity
        state: "{{ state_attr('sensor.neevotank_raw', 'TankCapacity') }}"
        unit_of_measurement: "gal"
        
      - name: "Propane Last Reading"
        unique_id: propane_last_reading
        state: "{{ state_attr('sensor.neevotank_raw', 'LastReadingDate') }}"
        device_class: timestamp
```

## Configuration Details

| Parameter | Value | Description |
|-----------|-------|-------------|
| `scan_interval` | 1440 minutes (24 hours) | How often the sensor updates |
| `method` | GET | HTTP method for API requests |
| `authentication` | basic | Uses HTTP Basic Authentication |
| `value_template` | `{{ value_json[0].Level }}` | Extracts the first device's level |

## Troubleshooting

### Sensor Shows "Unavailable"

1. Verify your Neevo credentials in `secrets.yaml` are correct
2. Check that your Neevo account is active and has tank monitoring enabled
3. Review Home Assistant logs for authentication errors: **Settings → System → Logs**

### Wrong Tank is Being Monitored

If you have multiple tanks, the configuration pulls data from the first device (`[0]`). To select a different tank:

```yaml
value_template: "{{ value_json[1].Level }}"  # Second tank
json_attributes_path: "$[1]"
```

### Sensor Not Updating

- The sensor updates every 24 hours by default
- You can force an update by restarting Home Assistant
- Check the `LastReadingDate` attribute to see when Neevo last received data

### Changing Update Frequency

To update more or less frequently, modify the `scan_interval` value (in minutes):

```yaml
scan_interval: 720  # Update every 12 hours
scan_interval: 60   # Update every hour (not recommended - may overload API)
```

> **Warning:** Setting scan_interval too low may result in API rate limiting.

## Automations

### Low Tank Alert

Create an automation to notify you when the tank is low:

```yaml
automation:
  - alias: "Low Propane Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.neevotank_raw
        below: 20
    action:
      - service: notify.mobile_app_your_phone
        data:
          title: "Low Propane"
          message: "Propane tank is at {{ states('sensor.neevotank_raw') }}%"
```

## Support

- **Neevo Support:** Contact Neevo for issues with your tank monitoring hardware or account
- **Home Assistant Community:** [Home Assistant Forums](https://community.home-assistant.io/)

## Contributing

Found a bug or have a suggestion? Please open an issue on GitHub!

## License

This configuration is provided as-is for personal use. Neevo and its API are property of their respective owners.

---

**Disclaimer:** This is an unofficial integration. Use at your own risk. Always maintain multiple methods of monitoring critical systems like propane levels.

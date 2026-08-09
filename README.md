# Marstek Cloud for Home Assistant

Home Assistant custom integration that connects your **Marstek battery** (Venus C/D/E) to Home Assistant via the **Marstek cloud API**, pulling live data and exposing it as sensor entities.

Because it uses the **cloud API** (instead of the local WLAN/UDP API), it stays reliable even when the battery's local Wi-Fi connection drops. The data is fetched straight from the Marstek cloud over your internet connection.

---

## ✨ Features

- **Automatic login & token refresh** — Logs in to the Marstek cloud API using your credentials, hashes your password (MD5) before sending, and automatically refreshes the token if it expires.
- **Configurable scan interval** — Set how often the integration polls the API (10–3600 seconds) during setup or later via the Options menu.
- **Battery metrics as sensors**
  - `soc` – State of charge (%)
  - `charge` – Charge power (W)
  - `discharge` – Discharge power (W)
  - `load` – Load power (W)
  - `profit` – Profit (€)
  - `version` – Firmware version
  - `sn` – Serial number
  - `report_time` – Timestamp of last report
  - `total_charge` – Total charge per device (kWh)
- **Cross-device total charge sensor** — `total_charge_all_devices`: sum of total charges across all batteries (kWh).
- **Diagnostic sensors** — `last_update`, `api_latency` (ms), `connection_status`.
- **Device registry integration** — Each battery appears as a device in HA with model, serial, firmware, and manufacturer.
- **Editable battery capacity** — Configure the default capacity (in kWh) per battery during setup or via Options.
- **Smart device filtering** — Automatically filters out non-compatible device types (e.g. `HME-3`).

---

## 🛠 Installation

### Via HACS (recommended)

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=bjornmonnens&repository=marstek_cloud&category=integration)

1. Open **HACS** in Home Assistant.
2. Click the **⋮** (three dots) in the top-right corner → **Custom repositories**.
3. Add this repository:
   ```
   https://github.com/bjornmonnens/marstek_cloud
   ```
   and set **Category** to **Integration**.
4. Click **Add**. **Marstek Cloud** will now appear in the HACS list.
5. Click **Install** (the `v0.2.0` release is downloaded) and **restart Home Assistant**.

### Manual copy

1. Drop the `custom_components/marstek_cloud` folder into your Home Assistant `custom_components` directory.
2. Restart Home Assistant.

---

## ⚙️ Setup / Configuration

1. Go to **Settings → Devices & Services → Add Integration** and search for **Marstek Cloud**.
2. Enter your **Marstek cloud email** and **password**, plus the desired **scan interval**.
3. The integration logs in, fetches your batteries, and creates the sensor entities.

After setup you can return to the integration's **Options** to:
- Change the **scan interval**.
- Set the **default battery capacity** (kWh) for each battery.

> **Note:** Default capacity is `5.12 kWh`. Minimum scan interval is 10 s, maximum 3600 s.

---

## 🧰 Entities

Each battery exposes the sensors `soc`, `charge`, `discharge`, `load`, `profit`, `version`, `sn`, `report_time`, `total_charge`, plus diagnostics `last_update`, `api_latency`, `connection_status`, and a cross-device `total_charge_all_devices`.

---

## 🔍 How it works (Logic Flow)

1. **Setup** — `config_flow.py` collects email, password, scan interval, capacities; stored securely in HA config entries.
2. **Coordinator & API** — `__init__.py` creates an `aiohttp` session, a `MarstekAPI` instance (cloud API), and a `MarstekCoordinator` (`DataUpdateCoordinator`) that schedules periodic updates.
3. **Login & token** — `MarstekAPI._get_token()` MD5-hashes the password, posts to the login endpoint, and stores the token. `get_devices()` calls the device-list endpoint with the token, refreshes on expiry, and clears the token on error code `8`.
4. **Data fetching** — The coordinator polls the API on the configured interval, filters ignored device types, and calculates latency.
5. **Entity creation** — `sensor.py` creates one sensor per metric plus diagnostics and the cross-device total.
6. **Updates** — Entities pull values from the coordinator's cached data on each refresh.

---

## 📡 API Endpoints Used

- **Login**
  ```
  POST https://eu.hamedata.com/app/Solar/v2_get_device.php?pwd=<MD5_PASSWORD>&mailbox=<EMAIL>
  ```
- **Get Devices**
  ```
  GET https://eu.hamedata.com/ems/api/v1/getDeviceList?token=<TOKEN>
  ```

---

## 🔧 Troubleshooting

- For verbose logging, add to `configuration.yaml`:
  ```yaml
  logger:
    logs:
      custom_components.marstek_cloud: debug
  ```
- If the data seems stale, check the configured **scan interval** under Options (the Marstek cloud API may also refresh data more slowly than requested).
- When reporting an issue, always attach the integration's **diagnostics export** and relevant HA logs.

---

The integration is a fork/continuation of [`DoctaShizzle/marstek_cloud`](https://github.com/DoctaShizzle/marstek_cloud).

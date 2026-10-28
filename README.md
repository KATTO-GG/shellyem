# Shelly EM Android App

Kotlin/Jetpack Compose client for the **Shelly EM** energy monitor via a Raspberry Pi server at `http://192.168.0.130:8090`.

## Features

- **Overview** — Live power, energy totals, cost estimate
- **Live** — 10-minute rolling line chart (channel 0 & 1)
- **Hourly** — Date-range stacked bar chart + daily summary list
- **Channels** — Editable names, meter health validation, reset history
- **Device** — Network, firmware, memory, storage, cloud status
- **Settings** — Base URL, auth token, electricity price, app lock, connection test

## Requirements

- Android 7.0+ (API 24)
- Raspberry Pi running the Shelly proxy server on **home Wi-Fi only**
- Shelly EM device configured on the same network

## Quick Start

1. Open project in Android Studio
2. Connect device/emulator on same Wi-Fi as Pi
3. Run `./gradlew assembleDebug` or press Play
4. Install `app/build/outputs/apk/debug/app-debug.apk`

## Configuration

### Base URL (Required)

Settings → **Pi Server URL** — default: `http://192.168.0.130:8090`

Must be the Pi's **LAN IP**. No internet/VPN/Tailscale — app only works on home Wi-Fi.

### Auth Token (Write Operations)

Required for: renaming channels, resetting meters.

1. Settings → **Enter Token**
2. Get token from Pi server admin
3. Token stored encrypted via `EncryptedSharedPreferences`
4. On 401: token cleared, re-prompted automatically

### Electricity Price

Settings → **Electricity Price** — default `0.30 NZD/kWh`  
Used for cost estimates on Overview & Hourly screens.

### App Lock (Optional)

Settings → **App Lock** — enables biometric/device credential prompt on app launch.

## Network Security

- Cleartext HTTP allowed **only** for `192.168.0.130` (see `res/xml/network_security_config.xml`)
- All other traffic blocked
- Connection timeout: 2s connect, 4s read

## Architecture

```
MVVM + StateFlow
├── ShellyRepository — OkHttp + kotlinx.serialization, single source of truth
├── ViewModels — Overview, Live, Hourly, Channels, Device, Settings
├── DataStore — baseUrl, pricePerKwh, appLockEnabled
└── EncryptedSharedPreferences — auth token
```

## API Endpoints (Pi Server)

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/live` | GET | No | Current power/energy |
| `/hourly` | GET | No | Hourly kWh for date range |
| `/channels/names` | GET | No | Channel custom names |
| `/channels/names` | POST | **Yes** | Rename channels |
| `/resets` | GET | No | Recent meter reset events |
| `/meter-health` | GET | No | Channel validity + totals |
| `/info` | GET | No | Device/network/firmware info |
| `/test-connection` | GET | No | Connectivity check |

## Assumptions

- Pi server runs the proxy API matching the above contract
- Shelly EM has 2 channels (0 and 1)
- Energy values reported in **Wh** (app converts to kWh)
- Timestamps in **Unix seconds** (UTC)
- Server returns `401` for missing/invalid token on write endpoints
- App never caches auth token in plaintext

## Build

```bash
./gradlew assembleDebug
# APK at app/build/outputs/apk/debug/app-debug.apk
```

## License

Private use only.
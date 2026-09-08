# Seed Rate Control (Android app)

Computes the seed-metering ratio `x` from field parameters and sends it to
the STM32F411 Blackpill over Bluetooth (HC-05, classic SPP), matching the
`X <value>\n` command already built into the firmware's `BT_ProcessLine()`.

## Formula (matches Book1.xlsx exactly)

```
seedsPerWheelRev = (PI * wheelDiameterCm) / seedSpacingCm
x = seedsPerWheelRev / holesInSeedPlate
```

## How to open

1. Open Android Studio -> **Open** -> select this `SeedRateControl` folder.
2. Let Gradle sync (it will download the Android Gradle Plugin / Kotlin
   plugin on first sync -- needs internet access).
3. Run on a physical Android phone (Bluetooth classic SPP does not work
   in the emulator -- you need a real device).

## Before using the app

1. **Pair the HC-05 first**, the normal Android way: Settings -> Bluetooth
   -> scan -> select HC-05 -> enter PIN (default is usually `1234` or
   `0000`, printed on most HC-05 breakout boards).
2. Open the app, tap **Refresh** to list paired devices, select the HC-05
   from the dropdown, tap **Connect**.
3. Enter seed spacing / wheel diameter / hole count, tap
   **Calculate Ratio (x)**.
4. Tap **Send Ratio to Blackpill** -- this writes `X <value>\n` over the
   Bluetooth link.

## Firmware side (already implemented)

The STM32 firmware (`pid_motor_control_x_ratio_bluetooth_stm32f411_freertos.c`)
already listens on USART1 (PA9/PA10, connected to the HC-05) and parses
lines starting with `X` to update the live setpoint ratio -- no firmware
changes are needed to use this app as-is.

## Optional: seeing live status in the app's log window

The app's read-loop will display anything the Blackpill sends back over
the *same* Bluetooth link. Right now, the firmware's `Debug_Print()`
function only writes to USART2 (the wired debug link), not to USART1
(Bluetooth). If you want live `x / SP / FB / DUTY` values to show up in
the app's log window too, mirror the same `HAL_UART_Transmit()` call to
`&huart1` inside `Debug_Print()` in the firmware.

## Permissions

The app requests:
- `BLUETOOTH_CONNECT` / `BLUETOOTH_SCAN` on Android 12+ (API 31+)
- `ACCESS_FINE_LOCATION` on Android 11 and below (required by the OS for
  classic Bluetooth device access, even though no location is actually
  used)

## Known limitations / things to harden before field use

- No retry/reconnect logic if the Bluetooth link drops mid-operation --
  currently you'd need to tap Connect again.
- No input validation beyond "must be a positive number" -- doesn't
  check the resulting `x` against a sane agronomic range before sending.
- No persistence -- entered values reset when the app restarts. Consider
  saving common seed profiles (e.g. "Maize", "Wheat") with SharedPreferences
  if you'll be switching crops often.

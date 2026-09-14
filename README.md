# Step Feeder - ESP32 Aquarium Feeder
An ESP32-based automatic aquarium feeder built around the auger screw and food container salvaged from a Juwel SmartFeed 2.0, running on [ESPHome](https://esphome.io/) with full Home Assistant Integration.

<img src="/media/step_feeder_1.jpeg" width="300" height="400"><img src="/media/step_feeder_2.jpeg" width="300" height="400"><img src="/media/step_feeder_3.jpeg" width="300" height="400"><img src="/media/step_feeder_4.jpeg" width="300" height="400">

## Table of Contents
- [Features](#features)
- [Hardware](#hardware)
- [Prerequisites](#prerequisites)
- [Build steps](#build-steps)
- [Calibration](#calibration)
- [Home Assistant Integration](#home-assistant-integration)
- [Troubleshooting](#troubleshooting)

## Features

- **Tested** - for 1 year before publishing this feeder
- **Servo-driven auger** - reuses the SmartFeed 2.0's screw mechanism to dispense food in precise portions
- **Onboard display** - ST7789V screen shows the current time, Wifi signal strength, feeder status, next scheduled feeding, amount, and feeding mode.
- **5 independent feeding schedules** - each with its own time, amount, and mode
- **Two feeding modes** - Normal and Extra, with separately tunable servo levels for each
- **Pause mode** - temporarily disable all scheduled feedings without losing the schedule
- **Auto-dim display** - brightness automatically adjusts between day and night levels on a configurable schedule
- **Home Assistant integration** - feed on demand via the `feed_now` action, fires an `esphome.fish_feed` event on every feeding with device/amount/mode/timestamp
- **Feeding history** - tracks total feedings and last/next feeding time, all exposed as Home Assistant entities

## Hardware

- 1 x ESP32S (esp-idf framework) Max size: `26.50 x 52.00mm`
- 1 x ST7789V SPI display (240x280) [Link](https://amzn.eu/d/0fz5knmF)
- 1 x MG90S
- 1 x Juwel SmartFeed 2.0
- 1 x USB cable
- 1 x USB power adapter
- 8 x jumper cable
- [3D Printed Parts](#3d-printed-parts)

## Prerequisites
- **ESPHome 2026.8.2 or newer** - the config uses `min_version: 2026.8.2`, so an older ESPHome install will refuse to compile it.
- **A `secrets.yaml` file** alongside the config, containing your WiFi credentials:
```yaml
wifi_ssid: "your-wifi-name"
wifi_password: "your-wifi-password"
```
- Multiple language variants of the config may exist (`step-feeder-(lang).yaml`). To create your own translation, copy the file and translate the `display_*` and entity-name substitutions at the top.

## Build steps

1. **Flash the step-feeder-(lang).yaml file.** Key settings to adjust in substitutions if needed:

| Setting | Purpose | Default |
|---|---|---|
| `name` | Internal device name (used in hostname etc.) | step-feeder |
| `friendly_name` | Friendly name shown in Home Assistant | Step Feeder |
| `timezone` | Your timezone (used for feeding schedule) | File specific |
| `servo_pin` | GPIO driving the feeding servo | GPIO18 |
| `servo_min_level` | Minimum PWM level for the servo | 2.5% |
| `servo_max_level` | Maximum PWM level for the servo | 24.5% |
| `initial_normal_level` | Default servo level for "normal" feed mode | 18 |
| `initial_extra_level` | Default servo level for "extra" feed mode | 28 |
| `initial_stop_level` | Servo level when stopped/idle | 5 |
| `display_scl_pin` | Display SPI clock pin (SCL/CLK) | GPIO14 |
| `display_sda_pin` | Display SPI data pin (SDA/MOSI) | GPIO23 |
| `display_res_pin` | Display reset pin | GPIO4 |
| `display_dc_pin` | Display data/command pin | GPIO26 |
| `display_cs_pin` | Display chip-select pin | GPIO27 |
| `display_blk_pin` | Display backlight pin | GPIO16 |


2. **Disassemble the Juwel SmartFeed 2.0** and set aside the parts shown below (all screws) - the food container, auger screw, and the 4 screws that originally secured the container to the housing.

<img src="/media/container.jpeg" width="300" height="200">

3. **Mount the food container** to the 3D-printed base using the original 4 screws. The 3D-printed auger screw gear must be inserted into the container **before** the container is screwed down - it can't be added afterward.

<img src="/media/container_screw.jpeg" width="300" height="200"><img src="/media/container_mount_1.jpeg" width="200" height="200"><img src="/media/container_mount_2.jpeg" width="200" height="200">

4. **Mount the MG90S servo** using its supplied screws, and route the servo cable through to the compartment where the ESP32 will sit.

<img src="/media/servo_screws.jpeg" width="100" height="200"><img src="/media/servo_mount.jpeg" width="200" height="200">

5. **Assemble the display**: mount the display to its 3D-printed base with 4 screws, connect the wiring, then feed the cable through the cable hole and attach the display cover. Note which jumper wire color corresponds to which pin - you'll need this when wiring up the ESP32.

<img src="/media/display.jpeg" width="300" height="300">

6. **Route the power cable through the hole and secure it with a zip tie.** 

<img src="/media/power_cable.jpeg" width="300" height="300">

7. **Wire the ESP32** to the servo, display, and backlight according to the pin table above, using the jumper-wire color mapping from step 5. Twist the positive and ground wires together and secure them with heat shrink, as shown in the picture.

<img src="/media/esp32.jpeg" width="300" height="300">

8. Drill an hole thru the original servo arm like this:

<img src="/media/servo_arm.jpeg" width="300" height="300">

9. **Connect ESP32 to Home Assistant** and run 1 feed using the `feed_now` call

10. **Mount the servo arm** at this precies angle like in picture.

<img src="/media/arm.jpeg" width="300" height="300">

11. **Mount the arm with 1 screw.**

<img src="/media/3d_arm.jpg" width="300" height="300"><img src="/media/3d_arm_mounted.jpg" width="300" height="300">

## Calibration
The servo levels may need to be tuned after assembly but the default values should work.
1. In Home Assistant, use the **Servo Control** number entity to manually jog the servo to find the current arm level where the auger turns exactly one dispensing cycle.
2. **Normal Level** - set this to the servo level that advances the auger by one tooth.
3. **Extra Level** - set this to the servo level that advances the auger by two teeth.
4. **Stop Level** - set this to the servo level that returns the mechanism to its rest position.

## Home Assistant Integration
### Actions:

| Action | Purpose |
|---|---|
| `esphome.<name>_feed_now` | Trigger an immediate feeding. Parameters: `amount` (int), `mode` (`NORMAL` or `EXTRA`), `override_pause_mode` (bool - feed even if Pause mode is on) |
| `esphome.<name>_reset_total_feed` | Reset the total feedings counter back to 0 |

Example call:
```yaml
action: esphome.step_feeder_feed_now
data:
  amount: 10
  mode: "NORMAL"
  override_pause_mode: false
```
Every feeding also fires an `esphome.fish_feed` event containing `device`,`amount`,`mode`,and `timestamp` data, which you can use in automations.

## 3D Printed Parts
Recommended to use **PETG** for all parts for long term durability.

## Troubleshooting
**Display colors are inverted** - Some ST7789V panels require `invert_colors: false` instead of the default `true`. If the display colors appear inverted, try changing this setting.

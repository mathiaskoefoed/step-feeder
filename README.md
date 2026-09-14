# Step Feeder - ESP32 Aquarium Feeder
An ESP32-based automatic aquarium feeder built around the auger screw and food container salvaged from a Juwel SmartFeed 2.0, running on [ESPHome](https://esphome.io/) with full Home Assistant Integration.

<img src="/media/step_feeder_1.jpeg" width="300" height="400"><img src="/media/step_feeder_2.jpeg" width="300" height="400"><img src="/media/step_feeder_3.jpeg" width="300" height="400"><img src="/media/step_feeder_4.jpeg" width="300" height="400">

## Features

- **Tested** - for 1 year before publishing this feeder

- **Servo-driven auger** - reuses the SmartFeed 2.0's screw mechanism to dispense food in precise portions
- **Onboard display** - ST7789V screen shows the current time, Wifi status/signal, and time for the next scheduled feeding
- **5 independent feeding schedules** - each with its own time, amount, and mode
- **Two feeding modes** - Normal and Extra, with separately tunable servo levels for each
- **Pause mode** - temporarily disable all scheduled feedings without losing the schedule
- **Auto-dim display** - brightness automatically adjusts between day and noght levels on a configurable schedule
- **Home Assistant integration** - feed on demand via the `feed_now` action, fires an `esphome.fish_feed` event on every feeding with amount/mode/timestamp
- **Feeding history** - tracks total feedings and last/next feeding time, all exposed as Home Assistant entities

## Hardware

- 1 x ESP32S (esp-idf framework) Max size: `26.50 x 52.00mm`
- 1 x ST7789V SPI display (240x280) [Link](https://amzn.eu/d/0fz5knmF)
- 1 x MG90S
- 1 x Juwel SmartFeed 2.0
- 1 x USB cable
- 1 x USB power adapter
- 8 x jumper cable
- 3D printed parts


## Configuration
All device-specific settings (pins, timezone, display text, entity names) are defined as `substitutions` at the top of `step-feeder-(lang).yaml`, so the same base config can be reused across devices without touching the logic below.

Key settings to adjust for your build:

| Setting | Purpose |
|---|---|
| `servo_pin` | GPIO driving the servo |
| `servo_min_level` / `servo_max_level` | Calibrate to your servo's usable range |
| `initial_normal_level` / `initial_extra_level` / `initial_stop_level` | Starting position for each mode |
| `display_*_pin` | SPI pins for the ST7789V display |
| `timezone` | Used for scheduling and the on-screen clock |

## Build steps
1. **Disassemble the Juwel SmartFeed 2.0** and set aside the parts shown below (all screws) - the food container, auger screw, and the 4 screws that originally secured the container to the housing.

<img src="/media/container.jpeg" width="300" height="200">

3. **Mount the food container** to the 3D-printed base using the original 4 screws. The 3D-printed auger screw must be inserted into the container **before** the container is screwed down - it can't be added afterward.

<img src="/media/container_screw.jpeg" width="300" height="200"><img src="/media/container_mount_1.jpeg" width="200" height="200"><img src="/media/container_mount_2.jpeg" width="200" height="200">

4. **Mount the MG90S servo** using its supplied screws, and route the servo cable through to the compartment where the ESP32 will sit.

<img src="/media/servo_screws.jpeg" width="100" height="200"><img src="/media/servo_mount.jpeg" width="200" height="200">

6. **Assemble the display**: mount the display to its 3D-printed base with 4 screws, connect the wiring, then feed the cable through the cable hole and attach the display cover. Note which jumper wire color corresponds to which pin - you'll need this when wiring up the ESP32.

<img src="/media/display.jpeg" width="300" height="300">

7. **Route cable** thru hole, and add a zip tie.

<img src="/media/power_cable.jpeg" width="300" height="300">

8. **Wire the ESP32** to the servo, display, and backlight per the pin table under [Configuration](#configuration), using the color mapping from step 4. Twist (+, ground) with heat shrink like in picture to the servo cables.

<img src="/media/esp32.jpeg" width="300" height="300">

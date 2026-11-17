# Home Assistant blueprints

Automation blueprints for my Home Assistant setup.

## Repository layout

```
blueprints/automation/          all automation blueprints
  motion_scene_day_low_night_rgb.yaml
  shelly_2pm.yaml
  zha_button_to_rgb_light.yaml
```

Keep every blueprint in `blueprints/automation/`. Home Assistant ignores YAML files
that are in `blueprints/` directly.

## How to use a blueprint

1. Copy the repository into your Home Assistant config folder.
2. Restart Home Assistant, or reload automations.
3. Create an automation and select the blueprint.
4. Fill the inputs. Use the examples below as a template.

The `path` value is relative to `blueprints/automation/`.

---

## 1. Motion scene with keep-on button (`motion_scene_day_low_night_rgb.yaml`)

**Summary.** Controls one dimmable light and one RGB light from a presence sensor and a
button. Two clock windows select the mode. The button sets a keep-on state. That state
holds the dimmable light on and starts a purple breathing effect on the RGB light.

| Mode or gesture | When it applies | Result |
| --- | --- | --- |
| Day | Inside the day window (default 06:00–19:00) | Dimmable light on at `day_brightness`, RGB light off |
| Low light | Inside the low-light window (default 06:00–22:00), not day | Dimmable light on at `low_brightness`, RGB light off |
| Night | Outside both windows | RGB light on at `night_brightness`, fixed red, dimmable light off |
| Presence cleared | Keep-on not active | Both lights off |
| Single press, keep-on off | Any time | Keep-on on: dimmable light stays on, RGB light breathes purple |
| Single press, keep-on on | Any time | Keep-on off: RGB light off, dimmable light off at the next presence clear |
| Home Assistant start | Keep-on active | Restores the keep-on state and the breathing effect |

The dimmable light keeps its brightness when keep-on sets it on and it is already on.
At night, keep-on sets the dimmable light to `low_brightness`, because no day value
applies. Set `off_hold` to 30–60 s if the sensor reports short clear blips.

**Requirements**

- One `input_boolean` helper for the keep-on state. Create it in Settings > Devices &
  services > Helpers > Toggle.
- The button must be an `event` entity, for example a Shelly input. Only the event type
  in `toggle_event_type` toggles the state. The default is `single_push`.
- Keep the automation in `mode: parallel`. The blueprint sets it. Do not set
  `mode: single`, or the breathing effect blocks every trigger while it runs.

**Useful inputs**

| Input | Default | Purpose |
| --- | --- | --- |
| `motion_sensor` | – | Binary sensor with motion, occupancy, or presence |
| `dimmable_light` | – | Dimmer for day, low-light, and keep-on mode |
| `rgb_light` | – | RGB light for night mode and for the breathing effect |
| `toggle_button` | – | Event entity of the keep-on button |
| `toggle_event_type` | `single_push` | Event type that toggles the keep-on state |
| `keep_on_helper` | – | `input_boolean` helper for the keep-on state |
| `day_window_start` / `day_window_end` | `06:00:00` / `19:00:00` | Day window |
| `day_brightness` | `75` | Day brightness in percent |
| `low_window_start` / `low_window_end` | `06:00:00` / `22:00:00` | Low-light window |
| `low_brightness` | `35` | Low-light brightness in percent, also used by keep-on at night |
| `night_brightness` | `50` | Night brightness in percent |
| `night_turn_off_dimmer` | `true` | Turn the dimmer off in night mode |
| `red_r`, `red_g`, `red_b` | `255`, `0`, `0` | Night colour |
| `breath_r`, `breath_g`, `breath_b` | `128`, `0`, `128` | Breathing colour (purple) |
| `breath_low` / `breath_high` | `10` / `100` | Breathing brightness range in percent |
| `breath_period` | `4` | Seconds for one full breath, up and down |
| `respect_existing` | `true` | Keep a manual setting of the active light |
| `off_hold` | `00:00:00` | Time clear before turn-off |
| `transition` | `1` | Transition time in seconds |
| `debug_logbook` | `false` | Write one logbook entry per run with the resolved values |

**Example**

```yaml
# helper
input_boolean:
  bathroom_keep_on:
    name: Bathroom — keep the light on
```

```yaml
- id: bathroom_motion_scene
  alias: "Bathroom — motion scene with keep-on button"
  use_blueprint:
    path: motion_scene_day_low_night_rgb.yaml
    input:
      motion_sensor: binary_sensor.bathroom_presence_sensor_1
      dimmable_light: light.bathroom_dimmer
      rgb_light: light.bathroom_rgb
      toggle_button: event.shelly_bathroom_input_0
      toggle_event_type: single_push
      keep_on_helper: input_boolean.bathroom_keep_on
      day_window_start: "06:00:00"
      day_window_end: "19:00:00"
      day_brightness: 75
      low_window_start: "06:00:00"
      low_window_end: "22:00:00"
      low_brightness: 35
      night_brightness: 50
      night_turn_off_dimmer: true
      red_r: 255
      red_g: 0
      red_b: 0
      breath_r: 128
      breath_g: 0
      breath_b: 128
      breath_low: 10
      breath_high: 100
      breath_period: 4
      respect_existing: true
      off_hold: "00:00:00"
      transition: 1
      debug_logbook: false
```

---

## 2. Shelly 2PM shutter (`shelly_2pm.yaml`)

**Summary.** Controls one shutter from the two inputs of a Shelly 2PM module.

| Gesture | Result |
| --- | --- |
| Single click DOWN | Close the shutter, or stop it when it moves |
| Single click UP | Open the shutter, or stop it when it moves |
| Double click DOWN | Close fully, then tilt the slats to `tilt_position` |

The stop check runs after `moving_check_delay`, so the cover state can settle.
The tilt step waits for the `closed` state, up to `close_timeout`.

**Useful inputs**

| Input | Default | Purpose |
| --- | --- | --- |
| `input_0_event` | – | Event entity of input 0 (DOWN) |
| `input_1_event` | – | Event entity of input 1 (UP) |
| `cover` | – | Cover entity |
| `tilt_position` | `100` | Slat tilt after a double click |
| `moving_check_delay` | `0.5` | Delay in seconds before the movement check |
| `close_timeout` | `00:01:10` | Maximum wait for the closed state |

**Example**

```yaml
- id: shutter_living_room
  alias: "Shutter — living room (Shelly 2PM)"
  mode: single
  use_blueprint:
    path: shelly_2pm.yaml
    input:
      input_0_event: event.shelly_living_room_input_0
      input_1_event: event.shelly_living_room_input_1
      cover: cover.living_room_shutter
      tilt_position: 100
      moving_check_delay: 0.5
      close_timeout: "00:01:10"
```

Create one automation for each Shelly 2PM module.

---

## 3. ZHA Zigbee switch — RGB light (`zha_button_to_rgb_light.yaml`)

**Summary.** Controls an RGB(W) light from a 2-button ZHA Zigbee switch.

| Gesture | Result |
| --- | --- |
| Click UP | Turn the light on |
| Click DOWN | Turn the light off |
| Hold UP | Brighten by `step` percent every `interval` seconds, up to 100% |
| Hold DOWN | Dim by `step` percent every `interval` seconds, down to 1% |
| Double click UP | Warm white at the current brightness |
| Double click DOWN | Configurable RGB colour (default red) at the current brightness |

The blueprint uses two `input_boolean` helpers to detect a double click. A single
click is delayed by `double_click_window`. When the light is off and you hold UP,
the dimming starts at 1%. The blueprint handles both `move_with_on_off` and `move`.

**Requirements**

- Create two `input_boolean` helpers, one for each button.
- The Zigbee switch must send the ZHA commands `on`, `off`, `move_with_on_off`,
  `move`, and `stop_with_on_off`.

**Useful inputs**

| Input | Default | Purpose |
| --- | --- | --- |
| `remote` | – | ZHA switch device |
| `light` | – | Light entity |
| `dbl_up_pending` | – | Helper for the UP double click |
| `dbl_down_pending` | – | Helper for the DOWN double click |
| `step` | `10` | Brightness step in percent |
| `interval` | `0.5` | Time between dim steps in seconds |
| `double_click_window` | `0.5` | Double-click window and single-click delay |
| `red_r`, `red_g`, `red_b` | `255`, `0`, `0` | Colour for a double click DOWN |
| `warm_white_kelvin` | `0` | Colour temperature for a double click UP, `0` = the warmest |

**Example**

```yaml
# helpers
input_boolean:
  bedroom_switch_dbl_up:
    name: Bedroom switch — UP double click pending
  bedroom_switch_dbl_down:
    name: Bedroom switch — DOWN double click pending
```

```yaml
- id: bedroom_switch_light
  alias: "Bedroom — Zigbee switch controls RGB light"
  mode: single
  use_blueprint:
    path: zha_button_to_rgb_light.yaml
    input:
      remote: 3f4c5d6e7f8091a2b3c4d5e6f7081920
      light: light.bedroom_rgb
      dbl_up_pending: input_boolean.bedroom_switch_dbl_up
      dbl_down_pending: input_boolean.bedroom_switch_dbl_down
      step: 10
      interval: 0.5
      double_click_window: 0.5
      red_r: 255
      red_g: 0
      red_b: 0
      warm_white_kelvin: 0
```

Use the device ID from **Settings → Devices & services → Devices**.


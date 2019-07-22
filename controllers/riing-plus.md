# Riing Plus Controller
---

## Common

| VID    | PID (start) | PID (end) |
|--------|-------------|-----------|
| 0x264a | 0x1fa5      | 0x1fb5    |

<br>

|  Name       | Description                                                                                |
|-------------|--------------------------------------------------------------------------------------------|
| STATUS_BYTE | Byte where `0xfc` means success and `0xfe` failure                                         |
| LED_COUNT   | Number of leds supported by a device connected to a port                                   |
| PORT        | Id of the port<br>Starts from 1 to the number of ports on the controller                   |
| RGB_MODE    | Byte value indicating which RGB mode to use<br>Check below for specific values             |
| SPEED       | Byte value indicating speed in percent<br>From 0 to 100<br>Speeds from 1 to 19 are ignored |
| COLOR       | 3 byte color `[g, r, b]`                                                                   |
| COLORS      | List of COLOR bytes `[g, r, b, g, r, b, ...]`                                              |

##### RGB_SPEED

| Name    | Value  |
|---------|--------|
| EXTREME | `0x00` |
| FAST    | `0x01` |
| NORMAL  | `0x02` |
| SLOW    | `0x03` |

##### RGB_MODE

| Name     | Value            | Description
|----------|------------------|------------------------------------------------|
| FLOW     | 0x00 + RGB_SPEED | `COLORS` not used                              |
| SPECTRUM | 0x04 + RGB_SPEED | `COLORS` not used                              |
| RIPPLE   | 0x08 + RGB_SPEED | Requires 1 `COLOR`                             |
| BLINK    | 0x0c + RGB_SPEED | Requires `COLORS` list with `LED_COUNT` colors |
| PULSE    | 0x10 + RGB_SPEED | Requires `COLORS` list with `LED_COUNT` colors |
| WAVE     | 0x14 + RGB_SPEED | Requires `COLORS` list with `LED_COUNT` colors |
| PER_LED  | 0x18             | Requires `COLORS` list with `LED_COUNT` colors |
| FULL     | 0x19             | Requires 1 `COLOR`                             |

## Commands

> Values enclosed in `<...>` means they are optional
>
> `Read Bytes` of each command starts with `[<REPORT_ID>, FIRST_WRITE_BYTE, SECOND_WRITE_BYTE]` 
> but are skipped to improve readability. `REPORT_ID` is always `0x00` and is optional depending on the hid library
> 
> `Write Bytes` of each command have to begin with the `REPORT_ID` (`0x00`) but it's skipped to improve readability. Might be optional depending on the hid library

| Name                 | Write Bytes                              | Read Bytes                             | Description                                                               |
|----------------------|------------------------------------------|----------------------------------------|---------------------------------------------------------------------------|
| Init                 | `[0xfe, 0x33]`                           | `STATUS_BYTE`                          | Initializes the controller                                                |
| Get Firmware Version | `[0x33, 0x50]`                           | `[MAJOR, MINOR, PATCH]`                | Gets controller firmware version<br>Returns 3 bytes that make the version |
| Save Profile         | `[0x32, 0x53]`                           | `STATUS_BYTE`                          | Saves the current `RGB_MODE` and `SPEED` to the controller memory         |
| Set Speed            | `[0x32, 0x51, PORT, 0x01, SPEED]`        | `STATUS_BYTE`                          | Sets speed on `PORT` to `SPEED`                                           |
| Set RGB              | `[0x32, 0x52, PORT, RGB_MODE, <COLORS>]` | `STATUS_BYTE`                          | Sets rgb on `PORT` to `RGB_MODE`<br>lightning mode with `COLORS`          |
| Get Data             | `[0x33, 0x51, PORT]`                     | `[PORT, UNKNOWN, SPEED, RPM_L, RPM_H]` | Get data for `PORT`<br>`RPM` is calculated as `RPM_H << 8 + RPM_L`        |

<br>

---

<br>

### Unknown commands
* Sets speed by RPM?<br>`[0x32, 0x51, PORT, 0x02, PWM_H, PWM_L]`<br>`STATUS_BYTE`

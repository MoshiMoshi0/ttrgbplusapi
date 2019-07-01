# About

This api is a collection of commands that allows for creation of custom controller software for TT RGB PLUS devices. 

# Controllers

| Name                  | VID    | PID (start) | PID (end) |
|-----------------------|--------|-------------|-----------|
| Riing Controller      | 0x264a | 0x1f41      | 0x1f51    |
| Riing Plus Controller | 0x264a | 0x1fa5      | 0x1fb5    |
| Riing Trio Controller | 0x264a | 0x2135      | 0x2145    |
| DPSG Controller       | 0x264a | 0x2329      | 0x2329    |
| hiyatek (Unreleased?) | 0x264a | 0x2199      | 0x22a9    |

# Commands

### Common:

> These commands and values apply to all controllers unless described in specific controller section

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

> These modes apply to all controllers unless described in specific controller section

| Name     | Value            | Description
|----------|------------------|------------------------------------------------|
| FLOW     | 0x00 + RGB_SPEED | `COLORS` not used                              |
| SPECTRUM | 0x04 + RGB_SPEED | `COLORS` not used                              |
| RIPPLE   | 0x08 + RGB_SPEED | Requires 1 `COLOR`                             |
| BLINK    | 0x0c + RGB_SPEED | Requires `COLORS` list with `LED_COUNT` colors |
| PULSE    | 0x10 + RGB_SPEED | Requires `COLORS` list with `LED_COUNT` colors |
| WAVE     | 0x14 + RGB_SPEED | Requires `COLORS` list with `LED_COUNT` colors |
| BY_LED   | 0x18             | Requires `COLORS` list with `LED_COUNT` colors |
| FULL     | 0x19             | Requires 1 `COLOR`                             |

<br>

---

<br>

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

##### Unknown commands
* Sets speed by RPM?<br>`[0x32, 0x51, PORT, 0x02, PWM_H, PWM_L]`<br>`STATUS_BYTE`
* Riing Trio?<br>`[0x32, 0x52, port, 0x4d, 0x03, 0x01]`
* `[0x33, 0x54, port]`

---

## Riing Controller

### Common:

|  Name       | Description                                                                                |
|-------------|--------------------------------------------------------------------------------------------|
| PORT_COUNT  | Numer of fans connected to the controller (unverified)                                     |
| COLOR       | 3 byte color `[r, g, b]`                                                                   |
| COLORS      | List of COLOR bytes `[r, g, b, r, g, b, ...]`                                              |

<br>

| Name                 | Write Bytes                              | Read Bytes                             | Description                                                               |
|----------------------|------------------------------------------|----------------------------------------|---------------------------------------------------------------------------|
| Set Speed            | `[0x32, 0x51, PORT, 0x03, SPEED]`        | `STATUS_BYTE`                          | Sets speed on `PORT` to `SPEED`                                           |
| Set RGB              | `[0x32, 0x52, PORT, RGB_MODE, <COLORS>]` | `[0xfe]` if next port is used, `[0x00]` if not used (unverified)                             | Sets rgb on `PORT` to `RGB_MODE`<br>lightning mode with `COLORS`          |
| Get Data             | `[0x33, 0x51, PORT]`                     | `[PORT, PORT_COUNT, SPEED, RPM_L, RPM_H]` | Get data for `PORT`<br>`RPM` is calculated as `RPM_H << 8 + RPM_L`        |

##### RGB_MODE

| Name     | Value | Description
|----------|-------|--------------------|
| FLOW     | 0x00  | `COLORS` not used  |
| FULL     | 0x01  | Requires 1 `COLOR` |

---

## Riing Trio Controller

| Name                 | Write Bytes                                                  | Read Bytes        | Description                                                                   |
|----------------------|--------------------------------------------------------------|-------------------|-------------------------------------------------------------------------------|
| Set RGB              | `[0x32, 0x52, PORT, 0x24, 0x03, CHUNK_ID, 0x00, COLORS]` | `STATUS_BYTE` | Sets rgb on `PORT` to `COLORS`<br>For Riing Trio fans the `COLORS` (30 colors, 3 zones, 12+12+6) list is split in 2 chunks (19+11)<br>For Riing Duo fans the `COLORS` (18 colors, 2 zones, 12+6) list is split in 2 chunks (18+0)<br>`CHUNK_ID` indicates the chunk number starting from 1 |

---

## DPSG Controller

| Name                 | Write Bytes                                  | Read Bytes                                     | Description                                                    |
|----------------------|----------------------------------------------|------------------------------------------------|----------------------------------------------------------------|
| Init                 | `[0xfe, 0x31]`                               | Null terminated ascii string<br>with PSU model | Initializes the controller                                     |
| Get Firmware Version | `--------`                                   | `--------`                                     | Not Supported?                                                 |
| Save Profile         | `--------`                                   | `--------`                                     | Not Supported?                                                 |
| Set Speed            | `[0x30, 0x41, 0x01]`<br>`[0x30, 0x41, 0x02]`<br>`[0x30, 0x41, 0x03]`<br>`[0x30, 0x41, 0x04, SPEED]` | `STATUS_BYTE` | Silent mode<br>Performance mode<br>Fan off<br>Set fan to `SPEED` |
| Set RGB              | `[0x30, 0x42, RGB_MODE, <COLORS>]`           | `STATUS_BYTE`                                  | Sets rgb to `RGB_MODE`<br>lightning mode with `COLORS`         |
| Get Data             | See `PSU_DATA` table below                   | See `PSU_DATA` table below                     | Get PSU value                                                  |

##### PSU_DATA

| Name          | Write Bytes     | Read Bytes                                          |
|---------------|-----------------|-----------------------------------------------------|
| VIN           |  `[0x31, 0x33]` | `[VALUE_L, VALUE_H]`                                |
| VVOut12       |  `[0x31, 0x34]` | `[VALUE_L, VALUE_H]`                                |
| VVOut5        |  `[0x31, 0x35]` | `[VALUE_L, VALUE_H]`                                |
| VVOut33       |  `[0x31, 0x36]` | `[VALUE_L, VALUE_H]`                                |
| VIOut12       |  `[0x31, 0x37]` | `[VALUE_L, VALUE_H]`                                |
| VIOut5        |  `[0x31, 0x38]` | `[VALUE_L, VALUE_H]`                                |
| VIOut33       |  `[0x31, 0x39]` | `[VALUE_L, VALUE_H]`                                |
| Temp          |  `[0x31, 0x3a]` | `[VALUE_L, VALUE_H]`                                |
| FanRpm        |  `[0x31, 0x3b]` | `[VALUE_L, VALUE_H]`                                |
| Model         |  `[0x31, 0x3d]` | Null terminated ascii string with PSU model         |
| Serial Number |  `[0x31, 0x3f]` | Null terminated ascii string with PSU serial number |

Value calculation pseudocode:
```
value = VALUE_H << 8 | VALUE_L;
exponent = (value & 0x7800) >> 11;
sign = (value & 0x8000) >> 15;
fraction = (value & 0x7ff);

if (sign == 1)
    exponent -= 16

result = Math.Pow(2.0, exponent) * fraction
```
```
WATTS = VVOut33 * VIOut33
```

```
EFFICIENCY = (int)((VVOut12 * VIOut12 + VVOut5 * VIOut5 + VVOut33 * VIOut33) / 10.0)
```

##### Unknown commands
* `[0x31, 0x41]` Save Profile?
* `[0x30, 0x43, 0x01]`
---
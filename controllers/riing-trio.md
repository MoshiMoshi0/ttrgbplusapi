# Riing Trio Controller

## Common

| VID    | PID (start) | PID (end) |
|--------|-------------|-----------|
| 0x264a | 0x2135      | 0x2145    |

<br>
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

## Commands

> Values enclosed in `<...>` means they are optional
>
> `Read Bytes` of each command starts with `[<REPORT_ID>, FIRST_WRITE_BYTE, SECOND_WRITE_BYTE]` 
> but are skipped to improve readability. `REPORT_ID` is always `0x00` and is optional depending on the hid library
> 
> `Write Bytes` of each command have to begin with the `REPORT_ID` (`0x00`) but it's skipped to improve readability. Might be optional depending on the hid library

| Name                 | Write Bytes                                                  | Read Bytes        | Description                                                                   |
|----------------------|--------------------------------------------------------------|-------------------|-------------------------------------------------------------------------------|
| Init                 | `[0xfe, 0x33]`                           | `STATUS_BYTE`                          | Initializes the controller                                                |
| Get Firmware Version | `[0x33, 0x50]`                           | `[MAJOR, MINOR, PATCH]`                | Gets controller firmware version<br>Returns 3 bytes that make the version |
| Save Profile         | `[0x32, 0x53]`                           | `STATUS_BYTE`                          | Saves the current `RGB_MODE` and `SPEED` to the controller memory         |
| Get Data             | `[0x33, 0x51, PORT]`                     | `[PORT, UNKNOWN, SPEED, RPM_L, RPM_H]` | Get data for `PORT`<br>`RPM` is calculated as `RPM_H << 8 + RPM_L`        |
| Set Speed            | `[0x32, 0x51, PORT, 0x01, SPEED]`        | `STATUS_BYTE`                          | Sets speed on `PORT` to `SPEED`                                           |
| Set RGB              | `[0x32, 0x52, PORT, RGB_MODE, 0x03, CHUNK_ID, 0x00, COLORS]` | `STATUS_BYTE` | Sets rgb on `PORT` to `COLORS`<br>For Riing Trio fans the `COLORS` (30 colors, 3 zones, 12+12+6) list is split in 2 chunks (19+11)<br>For Riing Duo fans the `COLORS` (18 colors, 2 zones, 12+6) list is split in 2 chunks (18+0)<br>`CHUNK_ID` indicates the chunk number starting from 1 |

### RGB_MODE

| Name     | Value | Description
|---------|-------|------------------------------------------------|
| PER_LED | 0x24  | Requires `COLORS` list with `LED_COUNT` colors |

<br>
<br>

---

<br>
<br>

### Unknown commands
* `[0x32, 0x52, port, 0x4d, 0x03, 0x01]`
* `[0x33, 0x54, port]`

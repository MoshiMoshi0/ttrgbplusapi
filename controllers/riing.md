# Riing Controller
---

## Common

| VID      | PID (start)   | PID (end)   |
|----------|---------------|-------------|
| `0x264a` | `0x1f41`      | `0x1f51`    |

<br>

|  Name       | Description                                                                                |
|-------------|--------------------------------------------------------------------------------------------|
| STATUS_BYTE | Byte where `0xfc` means success and `0xfe` failure                                         |
| PORT_COUNT  | Numer of fans connected to the controller                                                  |
| PORT        | Id of the port<br>Starts from 1 to the number of ports on the controller                   |
| RGB_MODE    | Byte value indicating which RGB mode to use<br>Check below for specific values             |
| SPEED       | Byte value indicating speed in percent<br>From 0 to 100<br>Speeds from 1 to 19 are ignored |
| COLOR       | 3 byte color `[r, g, b]`                                                                   |
| COLORS      | List of COLOR bytes `[r, g, b, r, g, b, ...]`                                              |

## Commands

> Values enclosed in `<...>` means they are optional
>
> `Read Bytes` of each command starts with `[<REPORT_ID>, FIRST_WRITE_BYTE, SECOND_WRITE_BYTE]` 
> but are skipped to improve readability. `REPORT_ID` is always `0x00` and is optional depending on the hid library
> 
> `Write Bytes` of each command have to begin with the `REPORT_ID` (`0x00`) but it's skipped to improve readability. Might be optional depending on the hid library

| Name                 | Write Bytes                              | Read Bytes                                          | Description                                                        |
|----------------------|------------------------------------------|-----------------------------------------------------|--------------------------------------------------------------------|
| Set Speed            | `[0x32, 0x51, PORT, 0x03, SPEED]`        | `STATUS_BYTE`                                       | Sets speed on `PORT` to `SPEED`                                    |
| Set RGB              | `[0x32, 0x52, PORT, RGB_MODE, <COLORS>]` | `[0xfe]` if next port is used, `[0x00]` if not used | Sets rgb on `PORT` to `RGB_MODE`<br>lightning mode with `COLORS`   |
| Get Data             | `[0x33, 0x51, PORT]`                     | `[PORT, PORT_COUNT, SPEED, RPM_L, RPM_H]`           | Get data for `PORT`<br>`RPM` is calculated as `RPM_H << 8 + RPM_L` |

### RGB_MODE

| Name     | Value   | Description        |
|----------|---------|--------------------|
| FLOW     | `0x00`  | `COLORS` not used  |
| FULL     | `0x01`  | Requires 1 `COLOR` |
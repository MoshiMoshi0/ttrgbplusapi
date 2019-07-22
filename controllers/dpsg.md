# DPSG Controller

## Common

| VID    | PID (start) | PID (end) |
|--------|-------------|-----------|
| 0x264a | 0x2329      | 0x2329    |

<br>

|  Name       | Description                                                                                |
|-------------|--------------------------------------------------------------------------------------------|
| STATUS_BYTE | Byte where `0xfc` means success and `0xfe` failure                                         |
| LED_COUNT   | Number of fan leds in the PSU                                                       |
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

| Name                 | Write Bytes                                  | Read Bytes                                     | Description                                                    |
|----------------------|----------------------------------------------|------------------------------------------------|----------------------------------------------------------------|
| Init                 | `[0xfe, 0x31]`                               | Null terminated ascii string<br>with PSU model | Initializes the controller                                     |
| Get Data             | See `PSU_DATA` table below                   | See `PSU_DATA` table below                     | Get PSU value                                                  |
| Set Speed            | `[0x30, 0x41, 0x01]`<br>`[0x30, 0x41, 0x02]`<br>`[0x30, 0x41, 0x03]`<br>`[0x30, 0x41, 0x04, SPEED]` | `STATUS_BYTE` | Silent mode<br>Performance mode<br>Fan off<br>Set fan to `SPEED` |
| Set RGB              | `[0x30, 0x42, RGB_MODE, <COLORS>]`           | `STATUS_BYTE`                                  | Sets rgb to `RGB_MODE`<br>lightning mode with `COLORS`         |

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
```cpp
value = VALUE_H << 8 | VALUE_L;
exponent = (value & 0x7800) >> 11;
sign = (value & 0x8000) >> 15;
fraction = (value & 0x7ff);

if (sign == 1)
    exponent -= 16

result = Math.Pow(2.0, exponent) * fraction
```
```cpp
WATTS = VVOut12 * VIOut12 + VVOut5 * VIOut5 + VVOut33 * VIOut33
```
```cpp
EFFICIENCY = efficiency_lut[(int)(WATTS / 10.0)]
```
> efficiency_lut is different for each psu model, exact lut values are currently unknown


##### Unknown commands
* `[0x31, 0x41]` Save Profile?
* `[0x30, 0x43, 0x01]`
---
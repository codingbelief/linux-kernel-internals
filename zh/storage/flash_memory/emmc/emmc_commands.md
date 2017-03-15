# eMMC Commands

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| | | | | |

## CMD0

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| GO_IDLE_STATE | bc | [31:0] 00000000 | None | Resets the Device to idle state |
| GO_PRE_IDLE_STATE | bc | [31:0] F0F0F0F0 | None | Resets the Device to pre-idle state |
| BOOT_INITIATION | - | [31:0]FFFFFFFA | None | Initiate alternative boot operation |

## CMD1

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| SEND_OP_COND | bcr | [31:0] OCR without busy | R3 | Asks Device, in idle state, to send its [Operating Conditions Register](./emmc_device_registers.html#ocr-register) contents in the response on the CMD line. |

## CMD2

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| ALL_SEND_CID | bcr | [31:0] stuff bits | R2 | Asks Device to send its CID number on the CMD line |

## CMD3

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| SET_RELATIVE_ADDR | bcr | [31:16] RCA, [15:0] stuff bits | R1 | Assigns relative address to the Device |

## CMD4

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| SET_DSR | bc | [31:16] DSR, [15:0] stuff bits | - | Programs the DSR of the Device |

## CMD5

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| SLEEP_AWAKE | ac | [31:16] RCA, [15]Sleep/Awake, [14:0] stuff bits | r1b | Toggles the Device between Sleep state and Standby state. |

## CMD6

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| SWITCH | ac | [31:26] Set to 0, [25:24] Access, [23:16] Index, [15:8] Value, [7:3] Set to 0, [2:0] Cmd Set | r1b  | Switches the mode of operation of the selected Device or modifies the EXT_CSD registers. |

## CMD7

| Abbr. | Type | Argument | Resp | Description |
| -- | -- | -- | -- | -- |
| SELECT/DESELECT_CARD | ac | [31:16] RCA, [15:0] stuff bits | r1/r1b  | Command toggles a device between the standby and transfer states or between the programming and disconnect states. |

## CMD8

| Abbr. | Type  | Argument | Resp | Description |
| -- | --  | -- | -- | -- |
| SEND_EXT_CSD | adtc | [31:0] stuff bits | - | Device sends its EXT_CSD register as a block of data. |




















































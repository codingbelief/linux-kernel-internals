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


















































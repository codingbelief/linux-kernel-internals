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
| SEND_OP_COND | bcr | [31:0] OCR without busy | R3 | Asks Device, in idle state, to send its [Operating Conditions Register](/emmc_device_registers.html#ocr-register) contents in the response on the CMD line. |





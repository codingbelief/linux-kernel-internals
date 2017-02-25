# eMMC 工作模式

## Overview

TODO：Add Pic

eMMC Device 在 Power On、HW Reset 或者 SW Reset 时，Host 可以触发 eMMC Boot，让 eMMC 进入 Boot Mode。在此模式下，eMMC Device 会将 Boot Data 发送给 Host，这部分内容通常为系统的启动代码，如 BootLoader。

如果 Host 没有触发 Boot 流程或者 Boot 流程完成后，eMMC Device 会进入 Device Identification Mode。在此模式下，eMMC Device 将进行初始化，Host 会为 eMMC Device 设定工作电压、协商寻址模式以及分配 RCA 设备地址。

Device Identification Mode 结束后，就会进入 Data Transfer Mode。在此模式下，Host 可以发起数据读写流程。

进入 Data Transfer Mode 后，Host 可以发起命令，让 eMMC Device 进入 Interrupt Mode。在此模式下，eMMC Device 会等待内部的中断事件，例如，写数据完成等。eMMC Device 在收到内部中断事件时，会向 Host 发送 Response，然后切换到 Data Transfer Mode，等待 Host 后续的数据读写命令。

## Boot Operation Mode

### Boot From eMMC Device

TODO：Add Boot States

在 Power On、HW Reset 或者 SW Reset 后，如果 eMMC Device  有使能 Boot Mode（即，寄存器位 BOOT_PARTITION_ENABLE (EXT_CSD byte [179]) 指定了启动分区），那么 Host 有两种方式可以让 eMMC Device 进入 Boot Mode，分别定义为 Original Boot 和 Alternative Boot，如下：

1. Original Boot：拉低 CMD 信号并保持不少于 74 个时钟周期
2. Alternative Boot：保持 CMD 信号为高电平，74 个时钟周期后，发送参数为 0xFFFFFFFA 的 [CMD0](./emmc_commands.html#cmd0) 命令

进入 Boot Mode 后，eMMC Device 会根据寄存器位 BOOT_PARTITION_ENABLE 的设定，从两个 Boot partitions 和 UDA 中选择一个分区读取大小为 128KB × BOOT_SIZE_MULT (EXT_CSD byte [226]) 的 Boot Data 通过 Data Lines 发送给 Host。

在 Boot Data 数据传输过程中，Host 可以打断数据传输，提前结束 Boot Mode，方法如下：

1. Original Boot：传输过程中，拉高 CMD 信号
2. Alternative Boot：传输过程中，发送参数为 0xF0F0F0F0 的 [CMD0](./emmc_commands.html#cmd0) 命令

> **NOTE:**  
> Host 发送参数为 0xF0F0F0F0 的 [CMD0](./emmc_commands.html#cmd0) 命令，可以让 eMMC Device 进行 SW Reset  
> Host 拉高 RST_n 信号可以触发 eMMC Device 进行 HW Reset

### Boot Acknowledge

如果寄存器位 BOOT_ACK (EXT_CSD byte [179]) 被设定为 1， eMMC Device 会在 Host 触发 Boot Mode 的 50 ms 内，在 DAT0 上发送一个 "010" Boot ACK 给 Host。

包含 Boot ACK 的时序图如下所示：

TODO：Add Original Boot Pic

TODO：Add Alternative Boot Pic

### Boot Bus 配置

EXT_CSD byte [177] BOOT_BUS_CONDITIONS 寄存器用于配置在 Boot Mode 时，数据传输的总线状态。

通过 BOOT_BUS_CONDITIONS 寄存器配置，在 Boot Mode 时，总线可以支持以下几种模式：

| Mode | Data Rate  | Bus Width | Frequency | Max Data Transfer (x8) |
| -- | --  | -- | -- | -- |
| Backward Compatible | Single  | x1, x4, x8 | 0-26 MHz | 26 MB/s |
| High Speed SDR | Single  | x1, x4, x8 | 0-52 MHz | 52 MB/s |
| High Speed DDR | Dual | x4, x8 | 0-52 MHz | 104 MB/s |

BOOT_BUS_CONDITIONS 寄存器还可以配置退出 Boot Mode 后，是复位还是保留当前总线配置。如果配置为复位，那么退出 Boot Mode 后，总线会被复位为 Backward Compatible SDR x1 模式，如果配置为保留，那么退出 Boot Mode 后，总线会保留 Boot Mode 时的总线模式。

> **NOTE:**  
> BOOT_BUS_CONDITIONS 寄存器为 nonvolatile 属性，配置内容掉电不会丢失。  
> 如果 eMMC Device 没有经过 Boot Mode，BOOT_BUS_CONDITIONS 寄存器不会改变总线模式。  
> 退出 Boot Mode 后，还可以通过 HS_TIMING 和 BUS_WIDTH 寄存器配置总线模式。

### Boot Data 更新

eMMC Device 在从厂商出货时，没有存储内容，也没有使能 Boot Mode。使用 eMMC Devcie 产品需要先通过其他的方式（例如，通过 USB、UART 等）启动一个下载系统，将 Boot Data 以及其他的系统数据写入到 eMMC 中，同时使能 Boot Mode 并设定 Boot Bus 模式。而后，产品才能从 eMMC Device 上启动软件系统。

Boot Data 的更新与其他数据的写入类似，更多的数据写入细节，请参考 Data Transfer Mode 小节。

## Device Identification Mode

如果 Host 没有触发 Boot 流程或者 Boot 流程完成后，eMMC Device 会进入 Device Identification Mode。

TODO：Add States 

eMMC Device 在退出 Boot Mode 后或者没使能 Boot Mode 时 Power On、HW Reset 或者 SW Reset 后，会进入 Device Identification Mode 的 Idle State。

在 **Idle State** 下，eMMC Device 会进行内部初始化，Host 需要持续发送 [CMD1](./emmc_commands.html#cmd1)命令，查询 eMMC Device 是否已经完成初始化，同时进行工作电压和寻址模式协商。

Host 发送的 [CMD1](./emmc_commands.html#cmd1) 命令的参数中，包含了 Host 所支持的工作电压和寻址模式信息，eMMC Device 在接收到这些信息后，会进行匹配。如果 eMMC Devcie 和 Host 所支持的工作电压和寻址模式不匹配，那么 eMMC Device 会进入 **Inactive State**。

eMMC Device 在接收到 [CMD1](./emmc_commands.html#cmd1) 命令后，会将 [OCR register](./emmc_device_registers.html#ocr-register) 的内容作通过 Response 返回给 Host，其中包含了 eMMC Device 是否完成初始化的标志位、设备工作电压范围 Voltage Range 和存储访问模式 Memory Access Mode 信息。

eMMC Device 完成初始化后，就会进入 **Ready State**。在该 State 下，Host 会发送 [CMD2](./emmc_commands.html#cmd2) 命令，获取 eMMC Device 的 CID。

CID，即 Device identification number，用于标识一个 eMMC Device。它包含了 eMMC Device 的制造商、OEM、设备名称、设备序列号、生产年份等信息，每一个 eMMC Device 的 CID 都是唯一的，不会与其他的 eMMC Device 完全相同。

eMMC Device 接收到 [CMD2](./emmc_commands.html#cmd2) 后，会将 127 Bits 的 [CID register](./emmc_device_registers.html#cid-register) 的内容通过 Response 返回给 Host。

发送完 CID 后，eMMC Device 接着就会进入 **Identification State**。而后，Host 会发送参数包含 16 Bits RCA 的 [CMD3](./emmc_commands.html#cmd3) 命令，为 eMMC Device 分配 RCA。

设定完 RCA 后，eMMC Devcie 就完成了 Devcie Identification，进入 Data Transfer Mode。

> **NOTE:**  
> 本节只描述了单个 eMMC Device 的 Devcie Identification 过程，多个 eMMC 的 Device Identification 过程与此类似，更多的细节可以参考 eMMC Spec。

### Voltage Range

eMMC Device 支持 3.3v 和 1.8v 两种工作电压模式。在 1.8v 模式下，eMMC Device 会更加的省电。

TODO：描述如何设定或者切换不同的工作电压模式

### Memory Access Mode

Memory Access Mode 决定了 eMMC Device 在响应 Host 的数据读写请求时，是如何访问内部存储器的。在 eMMC 标准中存在两种 Memory Access Mode：Byte Access Mode 和 Sector Access Mode。

在数据读写的 Command 中，Host 会将读写的地址 A 作为 Command 的参数发送给 eMMC Device，在 Byte Access Mode 下，eMMC Device 将从第 A 个 Byte 开始进行读写操作，而在 Sector Access Mode 下，eMMC Device 将会从第 A 个 Sector 开始进行读写操作，一个 Sector 的大小为 512 Bytes 或者 4 KBytes，更大的 Sector 支持更大容量的存储器访问。

使用 Byte Access Mode 更加的灵活高效，但是由于寻址位数的限制，不能访问超过 2GB 的存储内容。Sector Access Mode 则支持大容量存储的访问，其中 512 Bytes Sector 可以支持最大 256 GB 容量的存储访问，更大容量的需求则可以使用 4 KBytes Sector。

<!--
This patch provides handling of the two way handshake when SEND_OP_COND
(CMD1) is send to mmc card. It is necessary to inform eMMC card if the
host can work with high capacity cards (Jedec JESD84-A441, point 7.4.3).

The extra flag MMC_MODE_HC (high capacity) is added to indicate if the
host is capable of handling the high capacity eMMC cards.

According to eMMC spec v4.3, Section 6.1 says that greater than 2GB
density cards are sector addressable and less than 2GB are byte addressable.
But Section 7.3.3 says that OCR bit 30 needs to be used which access mode
the host must use for all its future transactions.

In mainline kernel the support for less than 2GB cards is by checking
the density of card.
If it is greater than 2GB it is assumed to be supporting sector addressing.

But actually we need to check the OCR value to determine the correct access mode
See patch (http://www.spinics.net/lists/linux-mmc/msg01466.html) by Philip.

Sector A sector has a size of 512 bytes, the same size as a sector in the FAT file system
under DOS.
Block A page is the minimum size unit for writing and reading. The size is configurable
(512, 1024, 2048 bytes), but normally the size is 512 bytes.

High capacity negotiation	For devices larger than 2GB, the addressing mechanism is switched from byte addressing to sector addressing.
-->

### RCA - Relative device Address 

RCA 是在 Devcie Identification 过程中，由 Host 分配的 16 Bits 的设备地址，主要用于 Data Transfer Mode 下进行数据传输时，选定具体要进行读写的 eMMC Devcie。

## Data Transfer Mode

eMMC cards need to occasionally spend some time cleaning up garbage and perform cache/buffer related operations which are strictly on the card side and do not involve the host. These operations are at various levels based on the importance/severity of the operation 1- Normal, 2- Important and 3 - Critical. If an operation is delayed for long it becomes critical and the regular read/write from host can be delayed or take more time than expected. 
To avoid such issues the MMC HW and core driver provide a framework which can check for pending background operations and give the card some time to clear up the same.

## Interrupt Mode

## 参考资料

1. [Embedded Multi-Media Card (e•MMC) Electrical Standard (5.1)](http://www.jedec.org/sites/default/files/docs/JESD84-B51.pdf)  [PDF]  
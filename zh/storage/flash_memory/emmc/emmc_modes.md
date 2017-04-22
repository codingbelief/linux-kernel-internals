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

RCA 是在 Devcie Identification 过程中，由 Host 分配的 16 Bits 的设备地址，主要用于 Data Transfer Mode 下进行通信时，选定具体要进行操作的 eMMC Devcie。

Host 分配的 RCA 通常从 1 开始递增，0 地址作为广播地址。eMMC Devcie 的 [RCA register](./emmc_device_registers.html#rca-register) 保存了 Host 分配的 RCA。

TODO：确认掉电重启后，RCA register 的值是否丢失。

## Data Transfer Mode

eMMC Device 完成 Device Identification 后，就会进入到 Data Transfer Mode 的 Standby State。

在 Standby State 时，Host 可以通过发送 [CMD5](./emmc_commands.html#cmd5) 命令，让 eMMC Devcie 进入 低功耗的 Sleep State，而后再发送 [CMD5](./emmc_commands.html#cmd5) 命令则可以让 eMMC Device 退出 Sleep State。

在 Standby State 时，Host 可以通过发送 [CMD7](./emmc_commands.html#cmd7) 命令，让 eMMC Devcie 进入 Transfer State，而后再发送 [CMD7](./emmc_commands.html#cmd7) 命令则可以让 eMMC Device 退出 Transfer State。

### Read Data

在 Transfer State 时，Host 可以发送以下的命令，触发数据读取流程：

| 命令 | 描述 |
|-|-|
|[CMD8](./emmc_commands.html#cmd8)| 读取 [EXT_CSD](./emmc_device_registers.html#extend-csd-register) 寄存器数据 |
|[CMD17](./emmc_commands.html#cmd17)| 从指定的地址开始，读取一个 Block 的数据 |
|[CMD18](./emmc_commands.html#cmd18)| 从指定的地址开始，读取多个 Block 的数据 |
|[CMD21](./emmc_commands.html#cmd21)| 读取 Tuning Block 的数据 |

eMMC Device 在接收到上述几个 CMD 时，就会进入 Sending-data State。在此 State 下，eMMC Device 会持续将数据发送给 Host，直到指定数量的数据 Block 传输完成或者接收到 Host 发送的 [CMD12](./emmc_commands.html#cmd12) 传输停止命令。eMMC Device 在停止发送数据后，会返回到 Transfer State。  

如果 Host 在发送 [CMD18](./emmc_commands.html#cmd18) 前，先发送一个设定需要读取的 Block Count 的 [CMD23](./emmc_commands.html#cmd23)。eMMC Device 在完成指定 Block Count 的数据发送后，就自动结束数据传输，不需要 Host 主动发送停止命令 [CMD12](./emmc_commands.html#cmd12)。  

如果 Host 没有发送设定需要读取的 Block Count 的 Command，发送 Multiple Block Read 的 Command 后，eMMC Device 会持续发送数据，直到 Host 发送 Stop Command 停止数据传输。  

> NOTE:
> 如果在发送 [CMD18](./emmc_commands.html#cmd18) 前，先发送 [CMD23](./emmc_commands.html#cmd23) 设定需要读取的 Block Count，那么 eMMC Device 会在发送完指定数量的 Block 后，自动停止发送数据。

### Write Data

在 Transfer State 时，Host 可以发送以下的命令，触发数据写入流程：

| 命令 | 描述 |
|-|-|
|[CMD24](./emmc_commands.html#cmd24)| 写入一个 Block 的数据 |
|[CMD25](./emmc_commands.html#cmd25)| 写入多个 Block 的数据 |
|[CMD26](./emmc_commands.html#cmd26)| 写入 [CID](./emmc_device_registers.html#cid-register) 寄存器值 |
|[CMD27](./emmc_commands.html#cmd26)| 写入 [CSD](./emmc_device_registers.html#csd-register) 寄存器值 |

> NOTE:  
> [CID](./emmc_device_registers.html#cid-register) 寄存器值通常是只能写一次，由厂家在生产时确定并写入
> [CSD](./emmc_device_registers.html#csd-register) 寄存器值的部分位则可以多次改写。

eMMC Device 在接收到上述几个 CMD 时，就会进入 Receive-data State，在此 State 下，eMMC Devcie 会持续从 Host 接收数据，并存储到内部的 Buffer 或者寄存器中。

如果 Host 在发送 [CMD25](./emmc_commands.html#cmd25) 前，先发送一个设定需要写入的 Block Count 的 [CMD23](./emmc_commands.html#cmd23)。eMMC Device 在完成指定 Block Count 的数据接收后，就自动结束数据传输，不需要 Host 主动发送停止命令 [CMD12](./emmc_commands.html#cmd12)。  

如果 Host 没有发送设定需要写入的 Block Count 的 Command，发送 Multiple Block Write 的 Command 后，eMMC Device 会持续接收数据，直到 Host 发送 Stop Command 停止数据传输。  

eMMC Device 在开始进行写入操作时，会先将接收到的数据存储在内部 Buffer 中，然后在后台将 Buffer 中的数据写入到 Flash 中。通常情况下，Host 发送数据的速度会比 eMMC 写入 Flash 的速度快，所以内部的 Buffer 会出现写满的状态，此时 eMMC Devcie 会将 DAT0 信号线拉低作为 Busy 信号。Host 收到 Busy 信号后，就会暂停发送数据，等到 eMMC Device 将 Buffer 中的数据处完一部分并解除 Busy 信号后，再重新发送数据。

当 eMMC Device 完成数据接收后，就会进入到 Programming State，将内部 Buffer 中剩余未写入的数据写入到 Flash 中。 在该 State 下，eMMC Device 会持续将 DAT0 拉低，作为 Busy 信号。如果在完成写入前，有收到新的写入命令，那么 eMMC Device 会立刻退回到 Receive-data State，进行数据接收；如果在完成写入前，没有收到新的写入命令，则会在完成写入后，退回到 Transfer State。

如果 eMMC Devcie 在 Programming State 时，还没有完成写入操作，就收到参数不等于自身 RCA 的 [CMD7](./emmc_commands.html#cmd7) 命令，那么 eMMC Device 会进入到 Disconnect State。在该 State 下，eMMC Device 会继续进行写入操作，写入完成后则进入到 Stand-by State。

如果 eMMC Device 在 Disconnect State 时，还没有完成写入操作，就收到参数等于自身 RCA 的 [CMD7](./emmc_commands.html#cmd7) 命令，那么 eMMC Devcie 会从新回到 Programming State。

### Packed Commands - Packed Write and Packed Read

在实际应用场景中，通常会对 eMMC Device 有很多随机数据读取和写入操作，这些随机读写的目标地址往往都不是连续的，每一个随机读写都需要通过一个独立的读写流程来实现。

在 eMMC 4.5 及以后的标准中，引入了 Packed Commands 机制，将多个地址不连续的数据写入请求封装到一个 Multiple Block Write 流程中，同时将多个地址不连续的数据读取请求封装的一个 Multiple Block Read 流程中，以此减少读写请求数量，提高数据读写的效率。

TODO：Add Packed Read and Packed Write pic

#### Packed Write

发起 Packed Write 流程时，首先 Host 端会需要发送 packed flag 置 1 的 [CMD23](./emmc_commands.html#cmd23) SET_BLOCK_COUNT 命令。其中，CMD23 中的 Block Count 参数为 Packed Command Header 和实际写入的数据所占 Block 的总数。  
然后 Host 再发送 [CMD25](./emmc_commands.html#cmd25) 命令给 eMMC Device，开始进行多个 Block 的数据写入。其中第 1 个（或者前 8 个） Block 数据为 Packed Command Header，它包含了各个写请求写入数据的起始地址和长度等信息。  
eMMC Devcie 在接收到数据后，会根据 Packed Command Header 的信息，将数据写入到指定的位置。

#### Packed Read

发起 Packed Read 流程时，首先 Host 端会需要发送 packed flag 置 1 的 [CMD23](./emmc_commands.html#cmd23) SET_BLOCK_COUNT 命令。其中，CMD23 中的 Block Count 参数为 Packed Command Header 所占 Block 的数量。  
然后 Host 再发送 [CMD25](./emmc_commands.html#cmd25) 命令给 eMMC Device，开始进行 1 个（或者 8 个） Block 的 Packed Command Header 数据发送。Packed Command Header 包含了各个读请求读取数据的起始地址和长度等信息。  
发送完 Packed Command Header 后，Host 会再发送一个 packed flag 置 1 的 [CMD23](./emmc_commands.html#cmd23) SET_BLOCK_COUNT 命令。其中，CMD23 中的 Block Count 参数为待读取数据的 Block 的数量。  
紧接着，Host 再发送 [CMD18](./emmc_commands.html#cmd18) 命令，开始进行多个 Block 的数据读取。eMMC Devcie 会解析接收到的 Packed Command Header，然后将指定的数据发送给 Host 端。

> **NOTE:**  
> Host 也可以不发送第二个 CMD23 命令，在这种情况下，Host 需要主动发送 Stop 命令，通知 eMMC Device 停止数据发送。

#### Packed Command Header

TODO：add packed command header pic

Packed Command Header 的格式如上图所示，其中 CMD23_ARG_x 指示了各个请求数据读取或者写入的 Block 数，CMDxx_ARG_x （CMD18 或者 CMD25）则指示了各个请求数据读取或者写入的起始位置。

当 DATA_SECTOR_SIZE[61] = 0x00 时，即 Data Sector Size 为 512 Bytes 时，Packed Command Header 占 1 个 Block 大小，当 DATA_SECTOR_SIZE[61] = 0x01 时，即 Data Sector Size 为 4 KBytes 时，Packed Command Header 占 8 个 Block 大小。

### Erase


<!--
eMMC cards need to occasionally spend some time cleaning up garbage and perform cache/buffer related operations which are strictly on the card side and do not involve the host. These operations are at various levels based on the importance/severity of the operation 1- Normal, 2- Important and 3 - Critical. If an operation is delayed for long it becomes critical and the regular read/write from host can be delayed or take more time than expected. 
To avoid such issues the MMC HW and core driver provide a framework which can check for pending background operations and give the card some time to clear up the same.
-->

## Interrupt Mode

## 参考资料

1. [Embedded Multi-Media Card (e•MMC) Electrical Standard (5.1)](http://www.jedec.org/sites/default/files/docs/JESD84-B51.pdf)  [PDF]  
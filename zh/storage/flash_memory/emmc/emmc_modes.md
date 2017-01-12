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
2. Alternative Boot：保持 CMD 信号为高电平，74 个时钟周期后，发送参数为 0xFFFFFFFA 的 CMD0 命令

进入 Boot Mode 后，eMMC Device 会根据寄存器位 BOOT_PARTITION_ENABLE 的设定，从两个 Boot partitions 和 UDA 中选择一个分区读取大小为 128KB × BOOT_SIZE_MULT (EXT_CSD byte [226]) 的 Boot Data 通过 Data Lines 发送给 Host。

在 Boot Data 数据传输过程中，Host 可以打断数据传输，提前结束 Boot Mode，方法如下：

1. Original Boot：传输过程中，拉高 CMD 信号
2. Alternative Boot：传输过程中，发送参数为 0xF0F0F0F0 的 CMD0 命令

> **NOTE:**  
> Host 发送参数为 0xF0F0F0F0 的 CMD0 命令，可以让 eMMC Device 进行 SW Reset  
> Host 拉高 RST_n 信号可以触发 eMMC Device 进行 HW Reset

### Boot Acknowledge

如果寄存器位 BOOT_ACK (EXT_CSD byte [179]) 被设定为 1， eMMC Device 会在 Host 触发 Boot Mode 的 50 ms 内，在 DAT0 上发送一个 "010" Boot ACK 给 Host。

包含 Boot ACK 的时序图如下所示：

TODO：Add Original Boot

TODO：Add Alternative Boot

### Boot Bus 配置
width
spee
data rate
reset

### Boot Data 更新


Original Boot
Alternative Boot
boot acknowledge
boot partition uda partition
eMMC 裸片不支持 Boot
如何更新 Boot 分区数据
WP
Boot bus width 

为什么需要 Alternative boot 模式？

## Device Identification Mode

### Voltage Range

eMMC Device 部分支持 3.3v 和 1.8v，1.8v 有功耗优势。

Dual-Voltage MultimediaCard (DV-MMC)
One of the first substantial changes in MMC was the introduction of dual-voltage cards that support operations at 1.8 V in addition to 3.3 V. Running at lower voltages reduces the card's power consumption, which is important in mobile devices. However, simple dual-voltage parts quickly went out of production in favour of MMCplus and MMCmobile which offer additional capabilities on top of dual-voltage support.

### Memory Access Mode

在数据读写的 Command 中，会将读写的地址作为 Command 的参数发送给 eMMC Device，这个地址可能是按 Byte 寻址的（Byte Access Mode，即从第几个 Byte 开始进行读写），也可能是按照 Sector 寻址的 （Sector Access Mode，即从第几个 Sector 开始进行读写）。

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

## Data Transfer Mode

eMMC cards need to occasionally spend some time cleaning up garbage and perform cache/buffer related operations which are strictly on the card side and do not involve the host. These operations are at various levels based on the importance/severity of the operation 1- Normal, 2- Important and 3 - Critical. If an operation is delayed for long it becomes critical and the regular read/write from host can be delayed or take more time than expected. 
To avoid such issues the MMC HW and core driver provide a framework which can check for pending background operations and give the card some time to clear up the same.

## Interrupt Mode


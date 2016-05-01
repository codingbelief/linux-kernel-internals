# eMMC 简介

eMMC 是 embedded MultiMediaCard 的简称。MultiMediaCard，即 MMC， 是一种闪存卡（Flash Memory Card）标准，它定义了 MMC 的架构以及访问　Flash Memory 的接口和协议。而 eMMC 则是对 MMC 的一个拓展，以满足更高标准的性能、成本、体积、稳定、易用等的需求。

eMMC 的整体架构如图片 1 所示：


eMMC 内部主要可以分为 Flash Memory、Flash Controller 以及 CPU Interface 三大部分，后续的章节将会对这三个部分进行详细介绍。

## Flash Memory

Flash Memory 是一种非易失性的存储器，通常在嵌入式系统中用于存放系统、应用和数据等，类似与 PC 系统中的硬盘。

与传统的硬盘相比，Flash Memory 具有质量轻、能耗低、体积小、抗震能力强等的优点，但也有不少局限性，如下所示：

1. 需要先擦除再写入
Flash Memory 写入数据时有一定的限制。它只能将当前为 1 的比特改写为 0，而无法将已经为 0 的比特改写为 1，只有在擦除的操作中，才能把整块的比特改写为 1。

2. 块擦除次数有限
Flash Memory 的每个数据块都有擦除次数的限制（十万到百万次不等），擦写超过一定次数后，该数据块将无法可靠存储数据，成为坏块。

为了最大化的延长 Flash Memory 的寿命，在软件上需要做块擦除均衡（wear leveling），通过分散写入、动态映射等手段均衡使用各个数据块。同时，软件还需要进行坏块管理（Bad Block Management，BBM），识别和标识坏块，不让坏块参与数据存储。（注：除了擦写导致的坏块外，Flash Memory 在生产过程也会产生坏块，即固有坏块。）

3. 读写干扰
由于硬件实现上的物理特性，Flash Memory 在进行读写操作时，有可能会导致邻近的其他比特发生位翻转，导致数据异常。这种异常可以通过重新擦除来恢复。Flash Memory 通常会使用 ECC 等算法进行错误检测和数据修正。

5. 电荷泄漏
存储在 Flash Memory 存储单元的电荷，如果长期没有使用，会发生电荷泄漏，导致数据错误。不过这个时间比较长，一般十年左右。此种异常是非永久性的，重新擦除可以恢复。

根据硬件物理实现上的不同，Flash Memory 主要可以分为 NOR Flash 和 NAND Flash 两类，主要的差异如下所示：

* NAND Flash 读取速度与 NOR Flash 相近，根据接口的不同有所差异；
* NAND Flash 的写入速度比 NOR Flash 快很多；
* NAND Flash 的擦出速度比 NOR Flash 快很多；
* NAND Flash 最大擦出次数比 NOR Flash 多；
* NOR Flash 支持片上执行，可以在上面直接运行代码；
* NOR Flash 软件驱动比 NAND Flash 简单；
* NOR Flash 可以随机按字节读取数据，NAND Flash 需要按块进行读取。
* 大容量下 NAND Flash 比 NOR Flash 成本要低很多，体积也更小；

（注：擦除 NOR Flash 时是以 64～128KB 的块进行的，执行一个写入 / 擦除操作的时间为 5s，擦除 NAND Flash 是以 8～32KB 的块进行的，执行相同的操作最多只需要 4ms。）


### NOR Flash

NOR Flash 根据与 CPU 端接口的不同，可以分为 Parallel NOR Flash 和 Serial NOR Flash 两类。
Parallel NOR Flash 可以接入到 Host 的 SRAM/DRAM Controller 上，支持片上执行。Serial NOR Flash 的成本比 Parallel NOR Flash 低，主要通过 SPI 接口与 Host 连接。

鉴于 NOR Flash 擦写速度慢，成本高等特性，NOR Flash 主要应用于小容量、内容更新少的场景，例如 PC 主板 BIOS、路由器系统存储等。

### NAND Flash

NAND Flash 根据每个存储单元内存储比特个数的不同，可以分为 SLC（Single-Level Cell）、MLC（Multi-Level Cell） 和 TLC（Triple-Level Cell） 三类。其中，在一个存储单元中，SLC 可以存储 1 个比特，MLC 可以存储 2 个比特，TLC 则可以存储 3 个比特。

NAND Flash 的一个存储单元内部，是通过不同的电压等级，来表示其所存储的信息的。在 SLC 中，存储单元的电压被分为两个等级，分别表示 0 和 1 两个状态，即 1 个比特。在 MLC 中，存储单元的电压则被分为 4 个等级，分别表示 00 01 10 11 四个状态，即 2 个比特位。同理，在 TLC 中，存储单元的电压被分为 8 个等级，存储 3 个比特信息。


### Flash Controller

### CPU Interfaces


## 附录
### 相关链接
[NOR NAND Flash Guide: Selecting a Flash Storage Solution](https://www.micron.com/~/media/documents/products/product-flyer/flyer_nor_nand_flash_guide.pdf)
[Common Flash Memory Interface](https://en.wikipedia.org/wiki/Common_Flash_Memory_Interface)
[Quick Guide to Common Flash Interface](https://www.spansion.com/Support/Application%20Notes/Quick_Guide_to_CFI_AN.pdf)
[MICRON NOR Flash Technology](https://www.micron.com/products/nor-flash)
[MICRON NAND Flash Technology](https://www.micron.com/products/nand-flash)
[Wiki：闪存](https://zh.wikipedia.org/wiki/%E9%97%AA%E5%AD%98)
[Wiki：Flash File System](https://en.wikipedia.org/wiki/Flash_file_system)
[Wear Leveling in Micron® NAND Flash Memory](https://www.micron.com/~/media/documents/products/technical-note/nand-flash/tn2961_wear_leveling_in_nand.pdf)
[Understanding Flash: The Flash Translation Layer](https://flashdba.com/2014/09/17/understanding-flash-the-flash-translation-layer/)
[谈NAND Flash的底层结构和解析](http://blog.sina.com.cn/s/blog_4b4b54da01016rx3.html)
[闪存基础](http://www.ssdfans.com/?p=45)

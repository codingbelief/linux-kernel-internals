# Flash Memory

Flash Memory 是一种非易失性的存储器，在嵌入式系统中用于存放系统、应用和数据等，在 PC 系统中，则用在固态硬盘以及主板 BIOS 中。

## Flash Memory 的主要特性

与传统的硬盘相比，Flash Memory 具有质量轻、能耗低、体积小、抗震能力强等的优点，但也有不少局限性，主要如下：

1. 需要先擦除再写入  
  Flash Memory 写入数据时有一定的限制。它只能将当前为 1 的比特改写为 0，而无法将已经为 0 的比特改写为 1，只有在擦除的操作中，才能把整块的比特改写为 1。

2. 块擦除次数有限  
  Flash Memory 的每个数据块都有擦除次数的限制（十万到百万次不等），擦写超过一定次数后，该数据块将无法可靠存储数据，成为坏块。  
  为了最大化的延长 Flash Memory 的寿命，在软件上需要做擦写均衡（Wear Leveling），通过分散写入、动态映射等手段均衡使用各个数据块。同时，软件还需要进行坏块管理（Bad Block Management，BBM），标识坏块，不让坏块参与数据存储。（注：除了擦写导致的坏块外，Flash Memory 在生产过程也会产生坏块，即固有坏块。）

3. 读写干扰  
  由于硬件实现上的物理特性，Flash Memory 在进行读写操作时，有可能会导致邻近的其他比特发生位翻转，导致数据异常。这种异常可以通过重新擦除来恢复。Flash Memory 通常会使用 ECC 等算法进行错误检测和数据修正。

4. 电荷泄漏  
  存储在 Flash Memory 存储单元的电荷，如果长期没有使用，会发生电荷泄漏，导致数据错误。不过这个时间比较长，一般十年左右。此种异常是非永久性的，重新擦除可以恢复。
  
## Flash Memory 类型

根据硬件上存储原理的不同，Flash Memory 主要可以分为 NOR Flash 和 NAND Flash 两类。 
主要的差异如下所示：

* NAND Flash 读取速度与 NOR Flash 相近，根据接口的不同有所差异；
* NAND Flash 的写入速度比 NOR Flash 快很多；
* NAND Flash 的擦除速度比 NOR Flash 快很多；
* NAND Flash 最大擦次数比 NOR Flash 多；
* NOR Flash 支持片上执行，可以在上面直接运行代码；
* NOR Flash 软件驱动比 NAND Flash 简单；
* NOR Flash 可以随机按字节读取数据，NAND Flash 需要按块进行读取。
* 大容量下 NAND Flash 比 NOR Flash 成本要低很多，体积也更小；

（注：NOR Flash 和 NAND Flash 的擦除都是按块块进行的，执行一个擦除或者写入操作时，NOR Flash 大约需要 5s，而 NAND Flash 通常不超过 4ms。）

### NOR Flash

NOR Flash 根据与 CPU 端接口的不同，可以分为 Parallel NOR Flash 和 Serial NOR Flash 两类。  
Parallel NOR Flash 可以接入到 Host 的 SRAM/DRAM Controller 上，所存储的内容可以直接映射到 CPU 地址空间，不需要拷贝到 RAM 中即可被 CPU 访问，因而支持片上执行。Serial NOR Flash 的成本比 Parallel NOR Flash 低，主要通过 SPI 接口与 Host 连接。

![](parallel_vs_serial.png)  
**图片： Parallel NOR Flash 与 Serial NOR Flash**

鉴于 NOR Flash 擦写速度慢，成本高等特性，NOR Flash 主要应用于小容量、内容更新少的场景，例如 PC 主板 BIOS、路由器系统存储等。

### NAND Flash

NAND Flash 根据每个存储单元内存储比特个数的不同，可以分为 SLC（Single-Level Cell）、MLC（Multi-Level Cell） 和 TLC（Triple-Level Cell） 三类。其中，在一个存储单元中，SLC 可以存储 1 个比特，MLC 可以存储 2 个比特，TLC 则可以存储 3 个比特。

NAND Flash 的一个存储单元内部，是通过不同的电压等级，来表示其所存储的信息的。在 SLC 中，存储单元的电压被分为两个等级，分别表示 0 和 1 两个状态，即 1 个比特。在 MLC 中，存储单元的电压则被分为 4 个等级，分别表示 00 01 10 11 四个状态，即 2 个比特位。同理，在 TLC 中，存储单元的电压被分为 8 个等级，存储 3 个比特信息。

![](slc_mlc_tlc.png)  
**图片： SLC、MLC 与 TLC**

NAND Flash 的单个存储单元存储的比特位越多，读写性能会越差，寿命也越短，但是成本会更低。Table 1 中，给出了特定工艺和技术水平下的成本和寿命数据。

**Table 1**  

|| SLC | MLC | TLC |
| -- | -- | -- | -- |
| 制造成本 | 30-35 美元 / 32GB | 17 美元 / 32GB | 9-12 美元 / 32GB |
| 擦写次数 | 10万次或更高 | 1万次或更高 | 5000次甚至更高 |
| 存储单元 | 1 bit / cell | 2 bits / cell | 3 bits / cell |
（注：以上数据来源于互联网，仅供参考）

相比于 NOR Flash，NAND Flash 写入性能好，大容量下成本低。目前，绝大部分手机和平板等移动设备中所使用的 eMMC 内部的 Flash Memory 都属于 NAND Flash。PC 中的固态硬盘中也是使用 NAND Flash。

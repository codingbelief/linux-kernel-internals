# Raw Flash - NOR and NAND


#### 1.2.1 NOR Flash

NOR Flash 根据与 CPU 端接口的不同，可以分为 Parallel NOR Flash 和 Serial NOR Flash 两类。  
Parallel NOR Flash 可以接入到 Host 的 SRAM/DRAM Controller 上，支持片上执行。Serial NOR Flash 的成本比 Parallel NOR Flash 低，主要通过 SPI 接口与 Host 连接。

![](parallel_vs_serial.png)  
**图片： Parallel NOR Flash 与 Serial NOR Flash**

鉴于 NOR Flash 擦写速度慢，成本高等特性，NOR Flash 主要应用于小容量、内容更新少的场景，例如 PC 主板 BIOS、路由器系统存储等。

#### 1.2.2 NAND Flash

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

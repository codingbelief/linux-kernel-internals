# eMMC 简介

eMMC 是 embedded MultiMediaCard 的简称。MultiMediaCard，即 MMC， 是一种闪存卡（Flash Memory Card）标准，它定义了 MMC 的架构以及访问　Flash Memory 的接口和协议。而 eMMC 则是对 MMC 的一个拓展，以满足更高标准的性能、成本、体积、稳定、易用等的需求。

eMMC 的整体架构如图片 1 所示：


eMMC 内部主要可以分为 Flash Memory、Flash Controller 以及 CPU Interface 三大部分，后续的章节将会对这三个部分进行详细介绍。

## Flash Memory

Flash Memory 是一种非易失性的存储器，通常在嵌入式系统中用于存放系统、应用和数据等，类似与 PC 系统中的硬盘。
根据硬件物理实现上的不同，Flash Memory 主要可以分为 NOR Flash 和 NAND Flash 两类，主要的差异如下所示：

* NAND Flash 读取速度与 NOR Flash 相近，根据接口的不同有所差异；
* NAND Flash 的写入速度比 NOR Flash 快很多；
* NAND Flash 的擦出速度比 NOR Flash 快很多；
* NAND Flash 最大擦出次数比 NOR Flash 多；
* NOR Flash 支持片上执行，可以在上面直接运行代码；
* NOR Flash 软件驱动比 NAND Flash 简单；
* 大容量下 NAND Flash 比 NOR Flash 成本要低很多，体积也更小；

（注：擦除 NOR Flash 时是以 64～128KB 的块进行的，执行一个写入 / 擦除操作的时间为 5s，擦除 NAND Flash 是以 8～32KB 的块进行的，执行相同的操作最多只需要 4ms。）

### NOR Flash

NOR Flash 根据与 CPU 端接口的不同，可以分为 Parallel NOR Flash 和 Serial NOR Flash 两类。
Parallel NOR Flash 可以接入到 Host 的 SRAM/DRAM Controller 上，支持片上执行。Serial NOR Flash 的成本比 Parallel NOR Flash 低，主要通过 SPI 接口与 Host 连接。

鉴于 NOR Flash 擦写速度慢，成本高等特性，NOR Flash 主要应用于小容量、内容更新少的场景，例如 PC 主板 BIOS、路由器系统存储等。

更多 NOR  Flash 相关的内容，可以参考附录中的相关链接。

### NAND Flash



内部架构
cpu 接口
Nand Flash 特性
  读写方式
  读写速度
  ECC
  Bad Block Management
  Wear-Leveling

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

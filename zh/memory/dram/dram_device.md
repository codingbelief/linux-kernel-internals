# DRAM Device

在前面的章节中，介绍了 DRAM Cell 和 Memory Array。
在此章节中，将以 SDR SDRAM 为例，描述 DRAM Device 与 Host 端的接口，以及其内部的其他模块，包括 Control Logic、IO、Row & Column Decoder 等。

## SDRAM Interface

SDR SDRAM 是 DRAM 的一种，它与 Host 端的硬件接口如下图所示：

TODO: Add sdram interface

总线上各个信号的描述如下表所示：

| Symbol | Type | Description |
| -- | -- | -- |
| CLK | Input | 从 Host 端输出的同步时钟信号 |
| CKE | Input | 用于指示 CLK 信号是否有效，SDRAM 会根据此信号进入或者退出 Power down、Self-refresh 等模式 |
| CS# | Input | Chip Select 信号 |
| CAS# | Input | Column Address Strobe，列地址选通信号 |
| RAS# | Input | Row Address Strobe， 行地址选通信号 |
| WE# | Input | Write Enable，写使能信号 |
| DQML | Input | 当进行写数据时，如果该 DQML 为高，那么 DQ[7:0] 的数据会被忽略，不写入到 DRAM |
| DQMH | Input | 当进行写数据时，如果该 DQMH 为高，那么 DQ[15:8] 的数据会被忽略，不写入到 DRAM |
| BA[1:0] | Input | Bank Address，用于选择操作的 Memory Bank |
| A[12:0] | Input | Address 总线，用于传输行列地址 |
| DQ[15:0] | I/O | Data 总线，用于传输读写的数据内容 |

### SDRAM Operations

Host 与 SDRAM 之间的交互都是由 Host 以 Command 的形式发起的。一个 Command 由多个信号组合而成，下面表格中描述了主要的 Command。

| Command | CS# | RAS# | CAS# | WE# | DQM | BA[1:0] & A[12:0] | DQ[15:0] |
| -- | -- | -- | -- | -- | -- | -- | -- |
| Active             | L | L | H | H | X | Bank & Row | X |
| Read               | L | H | L | H | L/H | Bank & Col | X |
| Write              | L | H | L | L | L/H | Bank & Col | Valid |
| Precharge          | L | L | H | L | X | Code | X |
| Auto-refresh       | L | L | L | H | X | X | X |
| Self-refresh       | L | L | L | H | X | X | X |
| Load Mode Register | L | L | L | L | X | REG Value | X |


#### Active

Active Command 会通过 BA[1:0] 和 A[12:0] 信号，选中指定 Bank 中的一个 Row，并打开该 Row 的 wordline。在进行 Read 或者 Write 前，都需要先执行 Active Command。

#### Read

Read Command 将通过 A[12:0] 信号，发送需要读取的 Column 的地址给 SDRAM。然后 SDRAM 再将 Active Command 所选中的 Row 中，将对应 Column 的数据通过 DQ[15:0] 发送给 Host。

Host 端发送 Read Command，到 SDRAM 将数据发送到总线上的需要的时钟周期个数定义为 CL。

#### Write

Write Command 将通过 A[12:0] 信号，发送需要写入的 Column 的地址给 SDRAM，同时通过 DQ[15:0] 将待写入的数据发送给 SDRAM。然后 SDRAM 将数据写入到 Actived Row 的指定 Column 中。

SDRAM 接收到最后一个数据到完成数据写入到 Memory 的时间定义为 tWR （Write Recovery）。

#### Precharge

在进行下一次的 Read 或者 Write 操作前，必须要先执行 Precharge 操作。（具体的细节可以参考 [DRAM Storage Cell](../dram_storage_cell.html) 章节）

Precharge 操作是以 Bank 为单位进行的，可以单独对某一个 Bank 进行，也可以一次对所有 Bank 进行。如果 A10 为高，那么 SDRAM 进行 All Bank Precharge 操作，如果 A10 为低，那么 SDRAM 根据 BA[1:0] 的值，对指定的 Bank 进行 Precharge 操作。

SDRAM 完成 Precharge 操作需要的时间定义为 tPR。

#### Auto-Refresh

DRAM 的 Storage Cell 中的电荷会随着时间慢慢减少，为了保证其存储的信息不丢失，需要周期性的对其进行刷新操作。

SDRAM 的刷新是按 Row 进行，标准中定义了在一个刷新周期内（常温下 64ms，高温下 32ms）需要完成一次所有 Row 的刷新操作。

为了简化 SDRAM Controller 的设计，标准还定义了 Auto-Refresh 机制，该机制要求 SDRAM Controller 在一个刷新周期内，发送 8192 个 Auto-Refresh Command，即 AR， 给 SDRAM。

SDRAM 每收到一个 AR，就进行 n 个 Row 的刷新操作，n = 总的 Row 数量 / 8192
同时，SDRAM 内部维护一个刷新计数器，每完成一次刷新操作，就将计数器更新为下一次需要进行刷新操作的 Row。

一般情况下，SDRAM Controller 会周期性的发送 AR，每两个 AR 直接的时间间隔定义为 tREFI = 64ms / 8192 = 7.8 us。

SDRAM 完成一次刷新操作所需要的时间定义为 tRFC, 这个时间会随着 SDRAM Row 的数量的增加而变大。

由于 AR 会占用总线，阻塞正常的数据请求，同时 SDRAM 在执行 refresh 操作是很费电，所以在 SDRAM 的标准中，还提供了一些优化的措施，例如 DRAM Controller 可以最多延时 8 个 tREFI 后，再一起把 8 个 AR 同时发出。

更多相关的优化可以参考《大容量 DRAM 的刷新开销问题及优化技术综述》文中的描述。

#### Self-Refresh

Host 还可以让 SDRAM 进入 Self-Refresh 模式，降低功耗。在该模式下，Host 不能对 SDRAM 进行读写操作，SDRAM 内部自行进行刷新操作保证数据的完整。

更多各个 Command 相关的细节，可以参考后续的 [DRAM Timing](../dram_timing.html) 章节。

### Address Mapping

SDRAM Controller 的主要功能之一是将 CPU 对指定物理地址的内存访问操作，转换为 SDRAM 读写时序，完成数据的传输。  
在实际的产品中，通常需要考虑 CPU 中的物理地址到 SDRAM 的 Bank、Row 和 Column 地址映射。下图是一个 32 位物理地址映射的一个例子：

TODO： Add address mapping

## SDRAM 内部结构

如图所示，DRAM Device 内部主要有 Control Logic、Memory Array、Decoders、Reflash Counter 等模块。在后续的小节中，将逐一介绍各个模块的主要功能。

TODO: DRAM Devcie

### Control Logic

Control Logic 的主要功能是解析 SDRAM Controller 发出的 Command，然后根据具体的 Command 做具体内部模块的控制，例如：选中指定的 Bank、触发 refresh 等的操作。

Control Logic 包含了 1 个或者多个 Mode Register。该 Register 中包含了时序、数据模式等的配置，更多的细节会在 [DRAM Timing](../dram_timing.html) 章节进行描述。 

### Row & Column Decoder

Row Decoder 的主要功能是将 Active Command 所带的 Row Address 映射到具体的 wordline，最终打开指定的 Row。同样 Column Decoder 则是把 Column Address 映射到具体的 csl，最终选中特定的 Column。

### Memory Array

Memory Array 是存储信息的主要模块，具体细节可以参考 [DRAM Memory Orgization](../dram_memory_orgization.html) 章节的描述。

### IO

IO 电路主要是用于处理数据的缓存、输入和输出。其中 Data Latch 和 Data Register 用于缓存数据，DQM Mask Logic 和 IO Gating 等则用于输入输出的控制。 

### Refresh Counter

Refresh Counter 用于记录下次需要进行 refresh 操作的 Row。在接收到 AR 或者在 Self-Refresh 模式下，完成 一次 refresh 后，Refresh Counter 会进行更新。

## 不同类型的 SDRAM

目前市面上在使用的 DRAM 主要有 SDR、DDR、LPDDR、GDDR 这几类，后续小节中，将对各种类型的 DRAM 进行简单的介绍。

### SDR 和 DDR

SDR（Single Data Rate） SDRAM 是第一个引入 Clock 信号的 DRAM 产品，SDR 在 Clock 的上升沿进行总线信号的处理，一个时钟周期内可以传输一组数据，一次称为 Single Data Rate。

DDR（Double Data Rate） SDRAM 

DDR 如何做到每一代的设计在速率上翻一倍：1. 2n-prefetch 架构 2. clock 速率翻倍

### LPDDRx

### GDDRx

## 参考资料

1. Memory Systems - Cache Dram and Disk
2. 大容量 DRAM 的刷新开销问题及优化技术综述 [PDF]


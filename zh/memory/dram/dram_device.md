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

在进行下一次的 Read 或者 Write 操作前，必须要先执行 Precharge 操作。（具体的细节可以参考 DRAM Storage Cell 章节）

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

Host 还可以让 SDRAM 进入 Self-Refresh 模式，以降低功能。在该模式下，Host 不能对 SDRAM 进行读写操作，SDRAM 内部自行进行刷新操作保证数据的完整。


### Address Mapping



## SDRAM 内部结构

SDR SDRAM 是 DRAM 的一种，相对于异步的 DRAM，它引入了一个参考时钟，降低了 DRAM Controller 实现的复杂度，

TODO: DRAM Devcie

如图所示，DRAM Device 内部主要有 Control Logic、Memory Array、Decoders、Reflash Counter 等模块。在后续的小节中，将逐一介绍各个模块的主要功能。

### Control Logic



### Memory Array

内存编址，地址到 row 和 column 的转换

### Data IO

In electronics, a flip-flop or latch is a circuit that has two stable states and can be used to store state information. 

### Reflash Counter

## 更多类型的 DRAM

Just as with standard SDRAM, each generation of LPDDR has doubled the internal fetch size and external transfer speed.

### SDR

### DDRx

DDR 如何做到每一代的设计在速率上翻一倍：1. 2n-prefetch 架构 2. clock 速率翻倍

### LPDDRx

### GDDRx

## 参考资料

1. Memory Systems - Cache Dram and Disk
2. 大容量 DRAM 的刷新开销问题及优化技术综述 [PDF]


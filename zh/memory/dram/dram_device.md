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

#### Write

Write Command 将通过 A[12:0] 信号，发送需要写入的 Column 的地址给 SDRAM。然后 SDRAM 再将 Active Command 所选中的 Row 中，将对应 Column 的数据通过 DQ[15:0] 发送给 Host。

#### Precharge

Precharge 操作是以 Bank 为单位进行的，可以单独对某一个 Bank 进行，也可以一次对所有 Bank 进行。

#### Refresh

##### Auto-refresh
##### Self-refresh

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

### LPDDRx

### GDDRx
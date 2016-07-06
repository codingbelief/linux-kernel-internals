# DRAM Device

在前面的章节中，介绍了 DRAM Cell 和 Memory Array。
在此章节中，将以 SDR SDRAM 为例，描述 DRAM Device 与 Host 端的接口，以及其内部的其他模块，包括 Control Logic、IO、Row & Column Decoder 等。

## SDRAM Interface

SDR SDRAM 是 DRAM 的一种，它与 Host 端的硬件接口如下图所示：

TODO: Add sdram interface

总线上各个信号的描述如下表所示：
| Symbol | Type | Description |
| -- | -- | -- |
| CLK | Input | 2:2 |
| CKE | Input | 2:3 |
| CS# | Input | 2:4 |
| CAS# RAS# WE# | Input | 2:5 |
| DQML DQMH | Input | 2:6 |
| BA[1:0] | Input | 2:6 |
| A[12:0] | Input | 2:6 |
| DQ[15:0] | 1:6 | 2:6 |

### SDRAM Operations

#### Active

#### Read / Write

#### Precharge

Precharge 操作是以 Bank 为单位进行的，可以单独对某一个 Bank 进行，也可以一次对所有 Bank 进行。

#### Reflash

##### Auto-reflash
##### Self-reflash

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
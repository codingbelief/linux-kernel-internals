# DRAM Memory Orgnization

在 [DRAM Storage Cell](./dram_storage_cell.html) 章节中，介绍了单个 Cell 的结构。在本章节中，将介绍 DRAM 中 Cells 的组织方式。

首先，为了更清晰的描述 Cells 的组织方式，我们先对上一章节中的 DRAM Storage Cell 进行抽象，最后得到新的结构图，如下：



## Memory Array




CSL 是否合并决定了有多少列和 data width

逐步构建一个 Memory Array，解释 bitline 太多和太少的情况。最终提供一个 memory array 抽象。

4 – 8 banks
16K rows / bank
1024 columns / row
4 – 16 bits / column

long bitlines cause very slow edge rates
very short bitlines Array efficiency goes down; wastes valuable silicon area

### Rows
定义：
A row is a group of storage cells that are activated together when a row activation command is sent to the DRAM device.

#### Row Size

### Columns
定义：
Column: Smallest addressable quantity of DRAM on chip


#### Column Size

## Memory Bank

主要用于增加容量？

Bank is a unit which can be active independently and has same data bus
width as external output bus width

 so the memory controller may send commands to only one bank at a time,


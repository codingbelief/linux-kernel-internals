# DRAM Memory Orgnization

在 [DRAM Storage Cell](./dram_storage_cell.html) 章节中，介绍了单个 Cell 的结构。在本章节中，将介绍 DRAM 中 Cells 的组织方式。

为了更清晰的描述 Cells 的组织方式，我们先对上一章节中的 DRAM Storage Cell 进行抽象，最后得到新的结构图，如下：

TODO：Add diagram

## Memory Array

DRAM 在设计上，会将所有的 Cells 以特定的方式组成一个 Memory Array。本小节将介绍 DRAM 中是如何 Cells 以 Memory Array 的方式组织起来的。

首先，我们在不考虑形式的情况下，最简单的组织方式，就是在一个 Bitline 上，挂接更多的 Cells，如下图所示：

TODO： Add diagram

在实际制造过程中，我们并不会无限制的在 Bitline 上挂接 Cells。因为 Bitline 挂接越多的 Cells，Bitline 的长度就会越长，也就意味着 Bitline 的电容值会更大，这会导致 Bitline 的信号边沿速率下降（电平从高变低或者从低变高的速率），最终导致性能的下降。为此，我们需要限制一条 Bitline 上挂接的 Cells 的总数，并将剩余的 Cells 挂接到其他的 Bitline 上去。

从 Cell 的结构图中，我们可以发现，在一个 Cell 的结构中，有两条 Bitline，它们在功能上是完全等价的，因此，我们可以把 Cells 分摊到不同的 Bitline 上，以减小 Bitline 的长度。然后，Cells 的组织方式就变成了如下的形式：

TODO：Add diagram

当两条 Bitline 都挂接了足够多的 Cells 后，如果还需要继续拓展，那么就只能增加 Bitline 了，增加后的结构图如下：

TODO：Add diagram

从图中我们可以看到，增加 Bitline 后，Sense Amplifier、Read Latch 和 Write Driver 的数量也相应的增加了，这意味着成本、功耗、芯片体积都会随着增加。由于这个原因，在实际的设计中，会优先考虑增加 Bitline 上挂接的 Cells 的数量，避免增加 Bitline 的数量，这也意味着，一般情况下 Wordline 的数量会比 Bitline 多很多。

图片呈现了一个由 16 个 Cells 组成的 Memory Array。其中的控制信号有 8 个 Wordline、2 个 CSL、2 个 WE，一次进行 1 个 Bit 的读写操，也就是可以理解为一个 8 x 2 x 1 的 Memory Array。

如果把 2 个 CSL 和 2 个 WE 合并成 1 个 CSL 和 1 个 WE，如下图所示。此时，这个 Memory Array 就有 8 Wordline、1 个 CSL、1 个 WE，一次可以进行 2 个 Bit 的读写操作，也就是成为了 8 x 1 x 2 的 Memory Array。

TODO：Add diagram

按照上述的过程，不断的增加 Cells 的数量，最终可以得到一个 m x n x w 的 Memory Array，如下图所示

TODO：Add diagram

其中，m 为 Wordline 的数量、n 为 CSL 和 WE 控制信号的数量、w 为一次可以进行读写操作的 Bits。

### Data Width

Memory Array 的 Data Width 是指对该 Array 进行一次读写操作所访问的 Bit 位数。这个位数与 CSL 和 WE 控制线的组织方式有关。

### Rows

DRAM Memory 中的一个 Row 本质上就是所有接在同一跟 Wordline 上的 Cells，如下图所示。

TODO：Add diagram

DRAM 在进行数据读写时，选中某一 Row，实质上就是控制该 Row 所对应的 Wordline，打开 Cells，并将 Cells 上的数据缓存到 Sense Amplifiers 上。

#### Row Size

一个 Row 的 Size 即为一个 Row 上面的 Cells 的数量。其中一个 Cell 存储 1 个 Bit 的信息，也就是说，Row Size 即为一个 Row 所存储的 Bit 位数。

### Columns

Column 是 Memory Array 中可寻址的最小单元。一个 Row 中有 n 个 Column，其中 n = Row Size / Data Width。下图是 Row Size 为 32，Data Width 为 8 时，Column 的示例。

TODO：Add diagram

#### Column Size

一个 Column 的 Size 即为该 Column 上所包含的 Cells 的数量，与 Data Width 相同。Column Size 和 Data Width 在本质上是一样的，也是与 CSL 和 WE 控制线的组织方式有关。

## Memory Bank



主要用于增加容量？

Bank is a unit which can be active independently and has same data bus
width as external output bus width

 so the memory controller may send commands to only one bank at a time,


4 – 8 banks
16K rows / bank
1024 columns / row
4 – 16 bits / column

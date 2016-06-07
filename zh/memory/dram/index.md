# DRAM


## DRAM Storage Cell

从原理层面上看，一个最简单的，存储一个 Bit 信息的 DRAM Storage Cell 的结构如下图所示：

![](dram_storage_cell.png)

由以下 4 个部分组成：

* Storage Capacitor，即存储电容，它通过存储在其中的电荷的多和少，或者说电容两端电压的高和低，来表示逻辑上的 1 和 0。
* Access Transistor，即访问晶体管，它的导通和截止，决定了允许或禁止对 Storage Capacitor 所存储的信息的读取和改写。
* Wordline，即字线，它决定了 Access Transistor 的导通或者截止。 
* Bitline，即位线，它是外界访问 Storage Capacitor 的唯一通道，当 Access Transistor 导通后，外界可以通过 Bitline 对 Storage Capacitor 进行读取或者写入操作。

从上面的结构图上分析，我们可以很容易的推测出 DRAM Storage Cell 的数据读写流程：

1. 读数据时，Wordline 设为逻辑高电平，打开 Access Transistor，然后读取 Bitline 上的状态
2. 写数据时，先把要写入的电平状态设定到 Bitline 上，然后打开 Access Transistor，通过 Bitline 改变 Storage Capacitor 内部的状态。

然而，在具体实现上，我们对 DRAM Storage Cell 进行读写时，会遇到以下的问题：

**外界的逻辑电平与 Storage Capacitor 的电平不匹配**
Storage Capacitor 在电容一定的情况下，存储的电荷越少，充放电的时间就会越短，也就意味着写入的性能更好。  
在实际的 DRAM 实现中，为了获取更好的写入性能，通常只会在 Storage Capacitor 存储少量的电荷。此时，当 Access Transistor 导通时，Bitline 上呈现的电压通常是要远远低于外界电路的逻辑电平。外界电路无法直接通过 Bitline 来读取Storage Capacitor 所存储的信息。

**进行一次读取操作后，Storage Capacitor 存储的电荷会变化**
在进行一次读取操作的过程中，Access Transistor 导通后，由于 Bitline 和 Storage Capacitor 端的电压不一致，会导致 Storage Capacitor 中存储的电荷量被改变。最终可能会导致在下一次读取操作过程中，无法正确的读出 Storage Capacitor 内存储的信息。

为解决上述的问题，DRAM 在设计上，引入了 Differential Sense Amplifier。

## Differential Sense Amplifier

TODO: Add Differential Sense Amplifier Block Diagram

Differential Sense Amplifier 包含 Sensing Circuit 和 Voltage Equalization Circuit 两个主要部分。它主要的功能就是将 Storage Capacitor 的电压转换为逻辑 1 或者 0 所对应的电压，并且呈现到 Bitline 上，同时，在完成一次读取操作后，通过 Bitline 将 Storage Capacitor 中的电荷恢复到读取之前的状态。

在后面的小节中，我们通过一次完整的数据读取过程，来了解 Differential Sense Amplifier 工作原理。

### Precharge

TODO: Add Precharge Diagram

在这个阶段，首先会通过控制 EQ 信号，让 Te1、Te2、Te3 晶体管处于导通状态，将 Bitline 和 /Bitline 线上的电压稳定在 Vref 上。然后进入到下一个阶段。

### Access

TODO: Add Access Diagram

经过 Rrecharge 阶段， Bitline 和 /Bitline 线上的电压已经稳定在 Vref 上了，此时，通过控制 Wordline 信号，将 Tc1 晶体管导通。由于 Storage Capacitor 中存储正电荷，Tc1 导通后，会将 Bitline 线上的电压拉升到 Vref+。然后进入到下一个阶段。 

### Sense





## RAM Array Structures























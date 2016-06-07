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

然而，在具体实现上，我们对 DRAM Storage Cell 进行读写时，会遇到一下的问题：

**外界的逻辑电平与 Storage Capacitor 的电平不匹配**
Storage Capacitor 在电容一定的情况下，存储的电荷越少，充放电的时间就会越短，也就意味着写入的性能更好。  
在实际的 DRAM 实现中，为了获取更好的写入性能，通常只会在 Storage Capacitor 存储少量的电荷。此时，当 Access Transistor 导通时，Bitline 上呈现的电压通常是要远远低于外界的逻辑电平。外界电路无法直接通过 Bitline

 Storage Capacitor 
**进行一次读取操作后，Storage Capacitor 存储的电荷会变化**

3. ，为了性能，电荷会少，电压会低。需要做一个转换的操作
2. 一次读取后，电荷会变化，需要做 restore 的操作

为解决写问题，引入了 Differential Sense Amplifier。

## Differential Sense Amplifier


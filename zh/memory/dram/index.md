# DRAM


## DRAM Storage Cell

从原理层面上看，一个最简单的，存储一个 Bit 信息的 DRAM Storage Cell 可以由以下几个部分组成：

* Storage Capacitor，即存储电容，它通过存储在其中的电量的多和少，或者说电容两端电压的高和低，来表示逻辑上的 1 和 0。
* Access Transistor，即访问晶体管，它的导通和截止，决定了允许或禁止对 Storage Capacitor 所存储的信息的读取和改写。
* Wordline，即字线，它决定了 Access Transistor 的导通或者截止。 
* BitlineB，即位线，

![](dram_storage_cell.png)
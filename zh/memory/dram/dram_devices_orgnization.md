# DRAM Devices Orgnization

1 个 CS 对应一个 Rank！  
Channel 如何区别？ 每个 Channel 上的控制信号是一样的？ 数据总线拼接到一起？   
如果是共用数据总线的，那么必然需要在控制信号上加以区分，以保证不会同时访问总线。  
理论上，channel 是拓展总线的  
如果是数据总线是拼接的，那么可以共用控制信号  
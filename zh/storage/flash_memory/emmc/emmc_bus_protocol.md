# eMMC 总线协议



## Boot operation 

### Boot State

eMMC 中定义了 Boot State，在 Power-up、HW reset 或者 SW reset 后，如果满足一定的条件，eMMC 就会进入该 State。进入 Boot State 的条件如下：

1. CMD 信号保持低电平不少于 74 个时钟周期时，会触发 Original boot operation，进入 Boot State。
2. 在 74 个时钟周期后，在 CMD 信号首次拉低或者 Host 发送 CMD1 之前，Host 发送参数为 0xFFFFFFFA 的 COM0时，会触发 Alternative boot operation，进入 Boot State。



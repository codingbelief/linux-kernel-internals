# eMMC 总线协议

## eMMC 总线接口

eMMC 总线接口定义如下图所示：  

![](emmc_host_interfaces.png)  

各个信号的描述如下：

**CLK**  

CLK 信号用于从 Host 端输出时钟信号，进行数据传输的同步和设备运作的驱动。  
在一个时钟周期内，CMD 和 DAT0-7 信号上都可以支持传输 1 个比特，即 SDR (Single Data Rate) 模式。此外，DAT0-7 信号还支持配置为 DDR (Double Data Rate) 模式，在一个时钟周期内，可以传输 2 个比特。  
Host 可以在通讯过程中动态调整时钟信号的频率（注，频率范围需要满足 Spec 的定义）。通过调整时钟频率，可以实现省电或者数据流控（避免 Over-run 或者 Under-run）功能。
在一些场景中，Host 端还可以关闭时钟，例如 eMMC 处于 Busy 状态时，或者接收完数据，进入 Programming State 时。

**CMD**  

CMD 信号主要用于 Host 向 eMMC 发送 Command 和 eMMC 向 Host 发送对于的 Response。Command 和 Response 的细节会在后续章节中介绍。

**DAT0-7**  

DAT0-7 信号主要用于 Host 和 eMMC 之间的数据传输。在 eMMC 上电或者软复位后，只有 DAT0 可以进行数据传输，完成初始化后，可配置 DAT0-3 或者 DAT0-7 进行数据传输，即数据总线可以配置为 4 bits 或者 8 bits 模式。（疑问：为什么提供这两种模式？什么情况下用不同的模式？）

**Data Strobe**  

Data Strobe 时钟信号由 eMMC 发送给 Host，频率与 CLK 信号相同，用于 Host 端进行数据接收的同步。Data Strobe 信号只能在 HS400 模式下配置启用，启用后可以提高数据传输的稳定性，省去总线 tuning 过程。

## eMMC 总线通讯模型



![](multi_block_read.png)


![](multi_block_write.png)

![](no_resp_or_data.png)

需要描述的内容：

1. Pin 定义
2. 总线通讯模型，基于消息的命令和响应模型
3. 寻址方式，设备操作方式
4. 总线模式，兼容模式、高速模式、HS200 模式、HS400 模式
5. 命令、响应、数据传输的详细格式
6. 总线的测试和 tuning



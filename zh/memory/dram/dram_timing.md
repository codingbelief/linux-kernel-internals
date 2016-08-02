# DRAM Timing

## Why we need timing
为什么需要有 Timing：
1. 当一个资源正在被使用时，如果后续的操作也需要用到该资源，那么就必须等待
2. 启动某一个资源时，本身就需要一定的时间
3. 有些延时是为了节省功耗，例如通过限制某些操作的频率，来控制 DRAM 的平均电流

## 时序图例

## Row Active Command
tRCD、tRAS
Row sense
Row restore



tRCD: Row Address to Column Address Delay
CL: CAS latency
tRP: Row Precharge Time
tRAS: Row Active Time
tRC : Row Cycle Time. tRC = tRAS + tRP.
tRRD: Row to Row Delay (different banks)

RAS : Row Address Strobe
CAS : Column Address Strobe
tWR : Write Recovery Time, the time that must elapse between the last write command to a row and precharging it. Generally, TRAS = TRCD + TWR.

CAS Latency (tCL) - This is the most important memory timing. CAS stands for Column Address Strobe. If a row has already been selected, it tells us how many clock cycles we'll have to wait for a result (after sending a column address to the RAM controller).

Row Address (RAS) to Column Address (CAS) Delay (tRCD) - Once we send the memory controller a row address, we'll have to wait this many cycles before accessing one of the row's columns. So, if a row hasn't been selected, this means we'll have to wait tRCD + tCL cycles to get our result from the RAM.

Row Precharge Time (tRP) - If we already have a row selected, we'll have to wait this number of cycles before selecting a different row. This means it will take tRP + tRCD + tCL cycles to access the data in a different row.

Row Active Time (tRAS) - This is the minimum number of cycles that a row has to be active for to ensure we'll have enough time to access the information that's in it. This usually needs to be greater than or equal to the sum of the previous three latencies (tRAS = tCL + tRCD + tRP).

每一个时间参数所代表的含义以及如何计算得到

auto precharge 有何作用？

[timings explication](http://www.tweakers.fr/timings.html)
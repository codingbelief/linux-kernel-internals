# DRAM Timing


tRCD: Row Address to Column Address Delay
CL: CAS latency
tRP: Row Precharge Time
tRAS: Row Active Time
tRC : Row Cycle Time. tRC = tRAS + tRP.
tRRD: Row to Row Delay (different banks)

RAS : Row Address Strobe
CAS : Column Address Strobe
tWR : Write Recovery Time, the time that must elapse between the last write command to a row and precharging it. Generally, TRAS = TRCD + TWR.


每一个时间参数所代表的含义以及如何计算得到

[timings explication](http://www.tweakers.fr/timings.html)
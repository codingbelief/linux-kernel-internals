# DRAM Timing

在 DRAM Device 章节中，我们简单介绍了 SDRAM 的 Active、Read、Write 等的操作，在本章节中，我们将详细的介绍各个操作的时序。

## Overview
如下图所示，SDRAM 的相关操作在内部大概可以分为以下的几个步骤：

1. Command Transport and Decode


TODO：内部流程图（1,2,3,4）


为什么需要有 Timing：

1. 当一个资源正在被使用时，如果后续的操作也需要用到该资源，那么就必须等待
2. 启动某一个资源时，本身就需要一定的时间
3. 有些延时是为了节省功耗，例如通过限制某些操作的频率，来控制 DRAM 的平均电流



## 时序图例

## Row Active Command
tRCD、tRAS
Row sense
Row restore

## Column Read Command
tCAS，即 CL
tBurst
burst length

## Column Write Command
tCWD
tWR

## Precharge Command
tRP
tRC

## Row Refresh Command
tREFI

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

auto precharge 有何作用？（每一个 row 的 access 都必须先进行 precharge）

[The following content are mainly got from Bruce Jacob’s book Memory Systems-Cache, DRAM, Disk(Chapter 11). Please refer to the book for more details.]

Row Access Commands (RA-Cmd):

Also called Row Activation Command, which is to move data from the cells in the DRAM arrays to the sense amplifiers (amp) and then restore the data back into the cells in the DRAM arrays as part of the same command.

tRCD: Row-Column(Command) Delay, the time for the RA-Cmd to move data from the DRAM cell arrays to the amp. After tRCD time from the assertion of the RA-Cmd, the entire row of activated data is held in the amp, and subsequent column read or column-write commands can then move data between the amp and the MC through the data bus.

{The time interval between row access and column access commands.}

tRAS: after tRCD time from the assertion of the RA-Cmd, data is available at the amp, but not yet fully restored to the DRAM cells. The time it takes for a RA-Cmd to discharge and restore data from the row of DRAM cells is known as tRAS (Row Access Strobe latency). After tRAS time from asseration of the RA-Cmd, the amp are assumed to have completed data restoration to the DRAM arrays, and the amp can then be precharged for another row access to a different row in the same bank of DRAM arrays.

{A bank cannot be precharged until at least tRAS time after the previous bank activation.}

Column-Read Commands (CR-Cmd):

A CR-Cmd moves data from the array of amp of a given bank through the data bus back to the MC.

tCAS/tCL: Column Access Strobe Latency, the time it takes for the DRAM device to place the requested data onto the data bus after issuance of the CR-Cmd.

tCCD: the time of minimum burst duration, or minimum column-to-column command timing. Determined by the prefetch length of the DRAM device.

tBURST: the duration of the data burst on the data bus for a single column-read command.

{The difference in the prefetch length of column access commands has limited impact on the generic memory-access protocol as long as tCCD < tBURST. In the case where tCCD > tBURST, the intra-bank column access commands are limited by tCCD rather than tBURST}

Column-Write Command (CW-Cmd):

Moves data from the MC to amp of the targeted bank.

tCWD: column write delay specifies the timing between assertion of the CW-Cmd on the command bus and the placement of the write data onto the data bus by the MC.

tWR: write recovery time, is the time it takes for the write data to propagate into the DRAM arrays, and it must be respected in the case of a precharge command that follows the write command.

tWTR: write-to-read turnaround time, accounts for the time that the I/O gating resources are released by the write command, and it must be respected in the case of a read command that follows the write command.

tRC: row-cycle time, tRC=tRAS+tRP. tRC denotes the minimum amount of time that a DRAM device needs to bring data from the DRAM cell arrays into the amp, restore the data to the DRAM cells, and precharge the bitlines to the reference voltage level for another row access command.

tAL: the posted CAS command is issued immediately after the row access command and the preset value of tAL defers the action of the column-read-and-precharge command to ensure that the column read aspect of the command does not violate the tRCD timing requirement.

tRTRS: consecutive read commands to different ranks of memory may not be issued and pipelined back to back depending on the system-level synchronization mechanism and the operating data rate of the memory system.

However, even with the availability of shared resources, considerations such as device current limitations can prohibit commands from being scheduled consecutively (that is tRRD and tFAW).

tRRD: Row-to-Row (Activation) Delay, defined in terms of nanoseconds instead of number of cycles; a minimum spacing between row activation is maintained regardless of operating data rates. In memory systems with two or more bank groups, consecutive row activation commands can be directed to different ranks to avoid the tRRD constraint.

tFAW: Four-bank Activation Window, specify a rolling time-frame in which a maximum of four row activations on the same DRAM device can be engaged concurrently. Row activation requests are spaced at least tRRD apart from each other and that the fifth row activation to a different bank is deferred until at least tFAW time period has passed since the first row activation was initiated.

For memory systems that implement the close-page row-buffer-management policy, tFAW places great constraints on the maximum sustainable bandwidth of a memory system with a single rank of memory regardless of operating data rates.


[timings explication](http://www.tweakers.fr/timings.html)
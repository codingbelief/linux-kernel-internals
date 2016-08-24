# DRAM Timing

在 DRAM Device 章节中，我们简单介绍了 SDRAM 的 Active、Read、Write 等的操作，在本章节中，我们将详细的介绍各个操作的时序。

## Overview

TODO：内部流程图（1,2,3,4）

如上图所示，SDRAM 的相关操作在内部大概可以分为以下的几个阶段：

1. Command transport and decode

    在这个阶段，Host 端会通过 Command Bus 和 Address Bus 将具体的 Command 以及相应参数传递给 SDRAM。SDRAM 接收并解析 Command，接着驱动内部模块进行相应的操作。

2. In bank data movement

    在这个阶段，SDRAM 主要是将 Memory Array 中的数据从 DRAM Cells 中读取到 Sense Amplifiers 中，或者将数据从 Sense Amplifiers 写入到 DRAM Cells。
 
3. In device data movement

    这个阶段中，数据将通过 IO 电路缓存到 Read Latchs 或者通过 IO 电路和 Write Drivers 更新到 Sense Amplifiers。

4. System data transport

    在这个阶段，进行读数据操作时，SDRAM 会将数据输出到数据总线上，进行写数据操作时，则是 Host 端的 Controller 将数据输出到总线上。

在上述的四个阶段中，每个阶段都会有一定的耗时，例如数据从 DRAM Cells 搬运到 Read Latchs 的操作需要一定的时间，因此在一个具体的操作需要按照一定时序进行。  
同时，由于内部的一些部件可能会被多个操作使用，例如读数据和取数据都需要用到部分 IO 电路，因此多个不同的操作通常不能同时进行，也需要遵守一定的时序。  
此外，某些操作会消耗很大的电流，为了满足 SDRAM 设计上的功耗指标，可能会限制某一些操作的执行频率。

基于上面的几点限制，SDRAM Controller 在发出 Command 时，需要遵守一定的时序和规则，这些时序和规则有相应的 SDRAM 标准定义。在后续的章节中，我们将对各个 Command 的时序进行详细的介绍。

## 时序图例

TODO：时序图例（需要描述各个部分的含义）

上图中，Clock 信号是由 SDRAM Controller 发出的，用于和 DRAM 之间的同步。在 DDRx 中，Clock 信号是一组差分信号，在本文中为了简化描述，将只画出其中的 Positive Clock。

Controller 与 DRAM 之间的交互，都是以 Controller 发起一个 Command 开始的。从 Controller 发出一个 Command 到 DRAM 接收并解析该 Command 所需要的时间定义为 tCMD，不同类型的 Command 的 tCMD 都是相同的。

DRAM 在成功解析 Command 后，就会根据 Command 在内部进行相应的操作。从 Controller 发出 Command 到 DRAM 执行完 Command 所对应的操作所需要的时间定义为 tParam。不同类型的 Command 的 tParam 可能不一样，相同 Command 的 tParam 由于 Command 参数的不同也可能会不一样。

Controller 发出一个 Command 后，必须要等到相应的 tParam 时间后，才能发出第二个 Command。

## Row Active Command

在进行数据的读写前，Controller 需要先发送 Row Active Command，打开 DRAM Memory Array 中的指定的 Row。Row Active Command 的时序如下图所示：

TODO：Row Active Command Timing

Row Active Command 可以分为两个阶段：

### Row Sense

Row Active Command 通过地址总线指明需要打开某一个 Bank 的某一个 Row。  
DRAM 在接收到该 Command 后，会打开该 Row 的 Wordline，将其存储的数据读取到 Sense Amplifiers 中，这一时间定义为 tRCD（RCD for Row Address to Column Address Delay）。  
DRAM 在完成 Row Sense 阶段后，Controller 就可以发送 Read 或 Write Command 进行数据的读写了。这也意味着，Controller 在发送 Row Active Command 后，需要等待 tRCD 时间才能接着发送 Read 或者 Write Command 进行数据的读写。  

### Row Restore

由于 DRAM 的特性，Row 中的数据在被读取到 Sense Amplifiers 后，需要进行 Restore 的操作（细节请参考 [DRAM Storage Cell](./dram_storage_cell.html) 章节）。Restore 操作可以和数据的读取同时进行，即在这个阶段，Controller 可能发送了 Read Command 进行数据读取。  
DRAM 接收到 Row Active Command 到完成 Row Restore 操作所需要的时间定义为 tRAS（RAS for Row Address Strobe）。  
Controller 在发出一个 Row Active Command 后，必须要等待 tRAS 时间后，才可以发起另一次的 Precharge 和 Row Access。

## Column Read Command

Controller 发送 Row Active Command 并等待 tRCD 时间后，再发送 Column Read Command 进行数据读取。  
Column Read Command 的时序如下图所示：

TODO：Column Read Command Timing

Column Read Command 通过地址总线指明需要读取的 Column 的起始地址。DRAM 在接收到该 Command 后，会将数据从 Sense Amplifiers 中通过 IO 电路搬运到数据总线上。  
DRAM 从接收到 Command 到第一组数据从数据总线上输出的时间称为 tCAS（CAS for Column Address Strobe），也称为 tCL（CL for CAS Latency），这一时间可以通过 mode register 进行配置，通常为 3~5 个时钟周期。  

DRAM 在接收到 Column Read Command 的 tCAS 时间后，会通过数据总线，将 n 个 Column 的数据逐个发送给 Controller，其中 n 由 mode register 中的 burst length 决定，通常可以将 burst length 设定为 2、4 或者 8。

开始发送第一个 Column 数据，到最后一个 Column 数据的时间定义为 tBurst。

## Column Write Command

Controller 发送 Row Active Command 并等待 tRCD 时间后，再发送 Column Write Command 进行数据写入。 
Column Write Command 的时序如下图所示：

TODO：Column Write Command Timing

Column Write Command 通过地址总线指明需要写入数据的 Column 的起始地址。Controller 在发送完 Write Command 后，需要等待 tCWD （CWD for Column Write Delay） 时间后，才可以发送待写入的数据。tCWD 在不同类型的 SDRAM 标准有所不同：

| Memory Type | tCWD |
| -- | -- |
| SDRAM | 0 cycles |
| DDR SDRAM | 1 cycle |
| DDR2 SDRAM | tCAS - 1 cycle |
| DDR3 SDRAM | programmable |

DRAM 接收完数据后，需要一定的时间将数据写入到 DRAM Cells 中，这个时间定义为 tWR（WR for Write Recovery）。

## Precharge Command

在 [DRAM Storage Cell](./dram_storage_cell.html) 章节中，我们了解到，要访问 DRAM Cell 中的数据，需要先进行 Precharge 操作。相应地，在 Controller 发送 Row Active Command 访问一个具体的 Row 前， Controller 需要发送 Precharge Command 对该 Row 所在的 Bank 进行 Precharge 操作。

下面的时序图描述了 Controller 访问一个 Row 后，执行 Precharge，然后再访问另一个 Row 的流程。

TODO：Precharge Command Timing

DRAM 执行 Precharge Command 所需要的时间定义为 tPR（PR for Precharge）。Controller 在发送一个 Row Active Command 后，需要等待 tRC（RC for Row Cycle）时间后，才能发送第二个 Row Active Command 进行另一个 Row 的访问。

从时序图上我们可以看到，tRC = tRAS + tPR，tRC 时间决定了访问 DRAM 不同 Row 的性能。在实际的产品中，通常会通过降低 tRC 耗时或者在一个 Row Cycle 执行多个数据读写等方式来优化性能。

> Note：  
> 在一个 Row Cycle 中，可以发起多个 Read 或者 Write Command 进行一个 Row 内的数据访问。这种情况下，由于不用进行 Row 切换，数据访问的性能会比需要切换 Row 的情况好很多。
> 在一些产品上，DRAM Controller 会利用这一特性，对 CPU 发起的内存访问进行调度，在不影响数据有效性的情况下，将同一个 Row 上的数据访问汇聚到一直起执行，以提供整体访问性能。

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
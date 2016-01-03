# QSPI 控制器 #

以将来的眼光设计的控制器，既支持现在的QSPI Flash，又支持现在市场上还没出现的将来可能出现的Flash颗粒以及他们可能使用的命令。

## 结构 ##

通过 Figure 22-1 和 Figure 22-2 了解 QSPI 控制器的结构。它由ZYNQ-7000里的 (Legacy) QSPI Controller 和一个新的通用 QSPI Controller 一起组成。Legacy QSPI Controller 的寄存器偏移地址和ZYNQ 7000的一模一样，这样老的程序甚至不用改代码。新的控制器使用0x100以后的偏移地址以及DMA使用0x800以后的地址。寄存器列表请参考[这里](http://www.xilinx.com/support/documentation/registers/ug1087/mod___qspi.html)

GQSPI_SEL[Generic_qspi_sel] 控制选择使用哪个控制器
- 1: 新的通用QSPI控制器
- 0: 老的QSPI 控制器

这个控制bit可以动态更改，前提是必须等现在正在进行的传输完成。

DMA和带延迟线的IO是两个控制器共享的。


### DMA–AXI Master ###
DMA从RX-FIFO中读取数据并发送到外部存储器比如DDR中。他是AXI Master。
DMA的运行状态从APB总线中的寄存器来监控。
DMA控制器接收一个QSPI控制器过来的总中断。
DMA控制器不支持unaligned data trasfer

### Generic Command FIFO ###

宽度20bit，深度32
用户通过APB写入FIFO

结合后面的Command章节一起阅读

### SPI Interface ###
时钟(SCLK)，数据，CS各两组

### Command Generator ###
读取Command FIFO和DATA FIFO后生成SPI波形

### RX FIFO ###
暂存接收到的数据
当RX FIFO满的时候，不会再产生SCLK，保证数据不丢失，FIFO不溢出。


## 通用QSPI控制器的Command ## 


## Generic FIFO Programming ##
### 配置 QSPI 模式 ###
看 Table 22-4 来理解 QSPI 模式寄存器配置。

控制位有4个参数。分别是：
- SPI Mode: 2'b01 = Single bit SPI, 2'b10 = Dual bit SPI, 2'b11 = Quad bit SPI
- Data Bus Select
- Receive
- Transmit

注意单bit模式时才能同时receive和transmit. 双bit和4bit的时候都是单向传输数据的。


### 配置数据传输长度 ###

组合使用 exponent 和普通 immediate data 方式。 exponent 为指数形式，表示2的N次方，并不能表达所有的长度整数。一个特定的长度可以拆成多个指数形式和一个或不需要 immediate data 形式的传输。

参考 Table 22-6 的例子来理解组合使用 exponent 和普通 immediate data 的例子。


### 配置 Poll 位###

由于 QSPI 设备的读写擦操作都比较慢，当需要不停查询 QSPI 状态的时候，可以使用 POLL 位来让控制器查询和比较，得到状态信息，而不需要由软件来手动查询和比较。查询得到预期的结果后可以产生中断。

一般流程是：
1. Controller 发送 Read Status Register Command (05h) 到 QSPI
1. Controller 不停地读 8 bits，用 POLL_DATA 的最后一个bit 和读取的值比较。Status Reg 的最后一个bit是 Write In Progress， 1 表示 busy.

相关寄存器是
- Generic FIFO
- POLL Register



#### POLL 中止 ####
Poll也可以中途中止，通过 POLL_TIMEOUT 和 EN_POLL_TIMEOUT 来控制 

-  EN_POLL_TIMEOUT = 0, POLL = 1: 不停地比较POLL_DATA(和Status比较还是和读回的数据比较？)
-  EN_POLL_TIMEOUT = 1, POLL_TIMEOUT = xxx, POLL = 1: 用一个（? Reference Clock）计数，直到Timeout发生。产生POLL_TIMEOUT_EXPIRE中断

如果用了QSPI Parallel，POLL就是比较两片位宽的数据。

### 配置 Stripe位 ###
Stripe可以控制在两个通道都使用了的情况下，TX FIFO中的数据发送到哪个通道。

- data_xfer = 0, stripe = 0: 把 immediate_data 同时发送给两个通道
- data_xfer = 1, stripe = 0: TX FIFO中偶数byte 发送给低位通道，奇数byte发送给高位通道
- data_xfer = 1, stripe = 1: TX FIFO中数据同时发送给两个通道

接收的时候stripe不能为1



### 发送奇数个Byte ###
只有在使用immediate_data模式的时候才能发送奇数个Byte (因为2的N次方的模式除了0次方只能表示偶数)。如果用immediate_data来发送奇数个byte，那最后一个byte是在低有效位。

## 操作模式 ##

Generic QSPI Controller 的操作模式分为 PIO 模式和 DMA 模式。PIO模式每次操作一个读写Command，DMA 模式可以用DMA连续读取大段数据，存放到 RX FIFO 中。注意DMA只能做读，不能做写。

### PIO 模式 ###

### DMA 模式 ###

### Flash Commands ###
各种对 Flash 的操作主要是通过控制发送给 Generic FIFO 的命令完成的。Table 22-12 和 Table 22-13 展示了两个例子。

### Write Protect ###

### Hold ###

### Interrupt ###


## 注意事项 ##

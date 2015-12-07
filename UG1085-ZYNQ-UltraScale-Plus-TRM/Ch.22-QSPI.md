# QSPI 控制器 #

以将来的眼光设计的控制器，既支持现在的QSPI Flash，又支持现在市场上还没出现的将来可能出现的Flash颗粒以及他们可能使用的命令。

## 结构 ##
由ZYNQ-7000里的QSPI Controller和一个新的通用QSPI Controller一起组成。（寄存器偏移地址一样，这样老的程序甚至不用改代码）

Generic_qspi_sel控制选择使用哪个控制器
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
Table 22-4
注意单bit模式时才能同时receive和transmit. 双bit和4bit的时候都是单向传输数据的。

### 配置数据传输长度 ###
组合使用exponent和普通immediate data, 因为exponent都是指数形式，并不能表达所有的长度整数。

### 配置 Poll 位###

Poll也可以停止，通过 POLL_TIMEOUT和EN_POLL_TIMEOUT 

### 配置 Stripe位 ###


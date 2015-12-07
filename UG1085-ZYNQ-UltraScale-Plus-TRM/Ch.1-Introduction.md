## Introduction to the UltraScale Architecture ##

UltraScale Architecture 包含的特性有：
- next-generation routing, 
- ASIC-like clocking, 
- 3D-on-3D ICs, 
- multiprocessor SoC (MPSoC) technologies, 
- power reduction features.

家族成员：
Virtex® UltraScale+: 16nm FinFET 最高性能和集成度
Virtex UltraScale: 20nm 很高的性能和集成度
Kintex® UltraScale+ : 16nm 下最好的价格/性能/功耗平衡
Kintex UltraScale : 20nm 下最好的价格/性能/功耗平衡
Zynq® UltraScale+ MPSoC 

## Technical Reference Manual Overview ##
ZYNQ MPSoC（以下简称MPSoC） 把PS和PL组合在一个芯片中。PS部分是ARM Cortex A53 64-bit 4核 APU 和 Cortex-R5 双核 RPU.

比起ZYNQ 7000的好处有
- Scalable PS with scaling for power and performance
- Low-power running mode and sleep mode
- Flexible user-programmable power and performance scaling
- Advanced configure system with device and user-security support
- Extended connectivity support including PCIe®, SATA, and USB 3.0
- Advanced user interface(s) with GPU and DisplayPort in the PS
- Increased DRAM and PS-PL bandwidth
- Improved memory traffic QoS
- Improved safety and reliability

适合的应用有blahblahblah…

## Block Diagram ##

PS PL的电源是分开的。PS启动可以不需要PL。PL只是用来扩展PS的功能。

MPSoC PS 部分有三个主要处理单元
- Cortex-A53 APU—ARM v8 架构 64-bit 4核.
- Cortex-R5 RPU—ARM v7 架构32-bit 双核RPU，带TCM.
- Mali-400 GPU—有 pixel and 
geometry 处理器，64KB L2 cache.

MPSoC PS部分有四组高速IO，支持以下协议
- PCIe v2.1
- SATA 3.1
- DisplayPort Source, 4k x 2k 分辨率
- SGMII

PS 部分MIO 78个，PS也可以通过EMIO接口使用PL的IO。

PMU(Platform Management Unit)管理内部模块的上电时序，由软件程序控制PS电轨上下电安全，执行逻辑自检，响应用户电源管理相关命令的请求。

CSU有BootROM，检测是否要做secure boot还是做non-secure boot. 做一些系统初始化，清除无用数据，读取Mode Pin，决定主启动设备，执行FSBL。（可以认为ZYNQ 7000的Boot ROM的主要功能现在整合在CSU ROM中）

系统重启后，PMU做一系列电源相关的初始化之后，CSU开始运行，从指定的启动设备上load FSBL。PS PL可以根据需要分别配置。JTAG可以控制PS和PL做配置和debug。

PL的电源可以关闭以节省功耗。PS的Power Island和模块的时钟都可以选择性关闭，或动态降频来进一步节省功耗。

Table 1-1总结了所有子模块的主要功能和特性


## System Software ##
Xilinx提供Standalone和Linux下的软件支持。



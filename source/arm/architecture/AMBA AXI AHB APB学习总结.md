## AMBA AXI AHB APB学习总结

### 一、概念介绍

#### 1、AHB（the Advanced High-performance Bus）
应用于高性能、高时钟频率的系统模块

#### 2、ASB（the Advanced System Bus）
是第一代AMBA系统总线，同AHB相比，它数据宽度要小一些，它支持的典型数据宽度为8位、16位、32位

#### 3、APB（the Advanced Peripheral Bus）
是本地二级总线（local secondary bus ），它主要是为了满足不需要高性能流水线接口或不需要高带宽接口的设备的互连

#### 4、AXI4
AXI4 协议是对 AXI3 的更新，在用于多个主接口时，可提高互连的性能和利用率。最多支持 256 位

#### 5、AXI4-Lite
AXI4-Lite 是 AXI4 协议的子协议，适用于与组件中更简单且更小的控件寄存器式的接口通信。AXI4-Lite 接口的主要功能如下：
- 所有事务的突发长度均为 1
- 所有数据存取的大小均与数据总线的宽度相同
- 不支持独占访问

#### 6、AXI4-Stream
- AXI4-Stream 协议可用于从主接口到辅助接口的单向数据传输，可显著降低信号路由速率。该协议的主要功能如下：
- 使用同一组共享线支持单数据流和多数据流
- 在同一互连内支持多个数据宽度
- FPGA 中实现的理想选择

#### 7、ACE4
ACE协议是在AXI4协议的基础上进行扩展，提供了对硬件一致性缓存的支持.

如下图所示是一个示例，每个master都有一个local cache，ACE协议会保证缓存的一致性.
> The ACE protocol permits cached copies of the same memory location to reside in the local cache of one or more master components.
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102120050427.png#pic_center)


#### 8、ACE5、ACE5-LiteDVM、ACE5-Lite、ACE5-LiteACP、AXI5、AXI5-Lit
同AXI4
### 二、实现上的介绍

#### 1、Access permissions (安全扩展等)
AMBA-AXI4总线的扩展, 增加了标志secure读和写地址线：AWPROT[1]和ARPROT[1]，用来标记Master的身份.
• **ARPROT[2:0]** defines the access permissions for read accesses.
• **AWPROT[2:0]** defines the access permissions for write accesses.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102113938190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

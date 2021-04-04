# TCR(Translation Control Register)寄存器详解



#### 1、TCR寄存器介绍

在ARM Core中(aarch64)，还有几个相关的系统寄存器:
- TCR_EL1
-  TCR_EL2
-  TCR_EL3
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013093543933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

比特位| 功能|说明|
|:-------- | :----- | :-----
|**ORGN1、IRGN1、ORGN0、IRGN0**|cache属性**|outer/inner cableability的属性(如直写模式、回写模式)
|**SH1、SH0**|cache的共享方式|cache的共享属性配置(如non-shareable, outer/inner shareable)
|**TG0/TG1**|Granule size|Granule size(其实就是页面的大小,4k/16k/64k)
|**IPS**|物理地址size|物理地址size,如32bit/36bit/40bit
|**EPD1、EPD0**|-|TTBR_EL1/TTBR_EL0的enable和disable
|**TBI1、TBI0**|-|top addr是ignore，还是用于MTE的计算
|**A1**|-|ASID的选择，是使用TTBR_EL1中的，还是使用TTBR_EL0中的
|**AS**|-|ASID是使用8bit，还是使用16bit

>- ORGN1 ：Outer cacheability attribute for memory associated with translation table walks using TTBR1_EL1
>- IRGN1 ：Inner cacheability attribute for memory associated with translation table walks using TTBR1_EL1
>- ORGN0 : Outer cacheability attribute for memory associated with translation table walks using TTBR0_EL1
>- IRGN0 : Iuter cacheability attribute for memory associated with translation table walks using TTBR0_EL1
>   <font color=blue size=4>0b00 Normal memory, Inner/Outer Non-cacheable.
0b01 Normal memory, Inner/Outer Write-Back Read-Allocate Write-Allocate Cacheable.
0b10 Normal memory, Inner/Outer Write-Through Read-Allocate No Write-Allocate Cacheable.
0b11 Normal memory, Inner/Outer Write-Back Read-Allocate No Write-Allocate Cacheable.</font>
><br>
>- SH0 : Shareability attribute for memory associated with translation table walks using TTBR0_EL1
>- SH1 : Shareability attribute for memory associated with translation table walks using TTBR1_EL1
><font color=blue size=4>0b00 Non-shareable.
>0b10 Outer Shareable.
>0b11 Inner Shareable.</font>
><br>
>- EPD0 : Translation table walk disable for translations using TTBR0_EL1
>- EPD1 : Translation table walk disable for translations using TTBR1_EL1
><font color=blue size=4>0 - enable
>1 - disable</font>
><br>
>- TBI0 : Top Byte ignored (TTBR0_EL1)
>- TBI1 : Top Byte ignored (TTBR1_EL1)
><font color=blue size=4>0b0 Top Byte used in the address calculation.
>0b1 Top Byte ignored in the address calculation</font>
><br>
>- T1SZ : The size offset of the memory region addressed by TTBR1_EL1. The region size is 2^(64-T1SZ) bytes
>- T0SZ : The size offset of the memory region addressed by TTBR0_EL1. The region size is 2^(64-T0SZ) bytes
><font color=blue size=4>bits [5:0]</font>
><br>
>- TG0 : Granule size for the TTBR0_EL1
>- TG1 : Granule size for the TTBR1_EL1
><font color=blue size=4>0b01 16KB.
>0b10 4KB.
>0b11 64KB.</font>
><br>
>- IPS : Intermediate Physical Address Size
><font color=blue size=4>0b000 32 bits, 4GB.
>0b001 36 bits, 64GB.
>0b010 40 bits, 1TB.
>0b011 42 bits, 4TB.
>0b100 44 bits, 16TB.
>0b101 48 bits, 256TB
>0b110 52 bits, 4PB</font>
><br>
>- A1 : Selects whether TTBR0_EL1 or TTBR1_EL1 defines the ASID
>0b0 TTBR0_EL1.ASID defines the ASID.
>0b1 TTBR1_EL1.ASID defines the ASID.
><br>
>- AS : ASID Size
><font color=blue size=4>0 - the upper 8 bits are ignored
>1 - the upper 16 bits are used for allocation and matching in the TLB</font>
><br>
>- HD : ARMv8.1-TTHM
>- HA : ARMv8.1-TTHM
>- HPD0 : ARMv8.1-HPD
>- HPD1 : ARMv8.1-HPD
>- HWU059 : ARMv8.2-TTPBHA
>- HWU060 : ARMv8.2-TTPBHA
>- HWU061 : ARMv8.2-TTPBHA
>- HWU062 : ARMv8.2-TTPBHA
>- HWU159 : ARMv8.2-TTPBHA
>- HWU160 : ARMv8.2-TTPBHA
>- HWU161 : ARMv8.2-TTPBHA
>- HWU162 : ARMv8.2-TTPBHA
>- TBID0 : ARMv8.3-PAuth
>- TBID1 : ARMv8.3-PAuth
>- NFD0 : SVE
>- NFD1 : SVE
>- E0PD0 : ARMv8.5-E0PD
>- E0PD1 : ARMv8.5-E0PD
>- TCMA0 : ARMv8.5-MemTag
>- TCMA1 : RMv8.5-MemTag
>
##### (1)、T1SZ、T0SZ -- 虚拟地址有效位
- T1SZ, bits [21:16] 通过TTBR1寻址的内存区域的大小偏移量，也就是TTBR1基地址下的一级页表的大小
- T0SZ, bits [5:0]

##### (2)、ORGN1、IRGN1、ORGN0、IRGN0 -- 内存的cacheability属性
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013175241972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
其实可以总结为这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013175443123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (3)、SH1、SH0 -- 内存的shareability属性
SH1, bits [29:28]
SH0, bits [13:12]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013184939895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
其实可以总结为这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013184959803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
Shareable的很容易理解，就是某个地址的可能被别人使用。我们在定义某个页属性的时候会给出。Non-Shareable就是只有自己使用。当然，定义成Non-Shareable不表示别人不可以用。某个地址A如果在核1上映射成Shareable，核2映射成Non-Shareable，并且两个核通过CCI400相连。那么核1在访问A的时候，总线会去监听核2，而核2访问A的时候，总线直接访问内存，不监听核1。显然这种做法是错误的。

对于Inner和Outer Shareable，有个简单的的理解，就是认为他们都是一个东西。在最近的ARM A系列处理器上上，配置处理器RTL的时候，会选择是不是把inner的传输送到ACE口上。当存在多个处理器簇或者需要双向一致性的GPU时，就需要设成送到ACE端口。这样，内部的操作，无论inner shareable还是outershareable，都会经由CCI广播到别的ACE口上。

##### (4)、TG0/TG1 - Granule size 页面的大小
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014112337111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (5)、IPS -- 中间物理地址有效位
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014121152796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (6)、EPD1、EPD0 是否开启TTBR0/TTBR1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014121854445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (7)、TBI1、TBI0 -- 虚拟地址的高位是否被忽略
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122151709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (8)、A1 - ASID使用TTBR0还是TTBR1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122307975.png#pic_center)
##### (10)、AS
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122355798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

**除了以上介绍的bit之外，剩余的bit都是特有功能使用或reserved的**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122503871.png#pic_center)

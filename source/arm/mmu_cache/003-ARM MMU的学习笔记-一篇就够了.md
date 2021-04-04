# ARM MMU的学习笔记-一篇就够了




### 一、ARMV8-aarch64的MMU
#### 1、MMU概念介绍
 **MMU分为两个部分**: TLB maintenance 和 address translation
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201025092840372.png#pic_center)



MMU的作用，主要是完成地址的翻译，无论是main-memory地址(DDR地址)，还是IO地址(设备device地址)，在开启了MMU的系统中，CPU发起的指令读取、数据读写都是虚拟地址，在ARM Core内部，会先经过MMU将该虚拟地址自动转换成物理地址，然后在将物理地址发送到AXI总线上，完成真正的物理内存、物理设备的读写访问


下图是一个linux kernel系统中宏观的虚拟地址到物理地址转换的视图，可以看出在MMU进行地址转换时，会依赖TTBRx_EL1寄存器指向的一个页表基地址.
其中，TTBR1_EL1指向特权模式的页表基地址，用于特权模式下的地址空间转换；TTBR0_EL0指向非特权模式的页表基地址，用于非特权模式下的地址空间转换.
刚刚我们也提到，CPU发出读写后, MMU会自动的将虚拟地址转换为为例地址，那么我们软件需要做什么呢？ 我们软件需要做的其实就是管理这个页表，按照ARM的技术要求去创建一个这样的页表，然后再将其基地址写入到TTBR1_EL1或TTBR0_EL1。当然，根据实际的场景和需要，这个基地址和页表中的内容都会发生动态变化. 例如，两个user进程进行切换时，TTBR0_EL1是要从一个user的页表地址，切换到另外一个user的页表地址。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101309232562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

#### 2、MMU地址翻译的过程
using a 64KB granule with a 42-bit virtual address space,地址翻译的过程(只用到一级页表的情况)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101313293157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

使用二级页表的情况举例：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013132956407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
#### 3、在secure和non-secure中使用MMU
TTBRx_EL1是banked的，在linux和optee双系统的环境下，可同时开启两个系统的MMU.
在secure和non-secure中使用不同的页表.secure的页表可以映射non-secure的内存，而non-secure的页表不能去映射secure的内存，否则在转换时会发生错误
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013133302768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
#### 4、在不同异常等级中使用MMU
在ARMV8-aarch64架构下，页表基地址寄存器有:
- TTBR0_EL1  <font color=blue size=3>-- banked</font>
- TTBR1_EL1  <font color=blue size=3>-- banked</font>
- TTBR1_EL2
- TTBR1_EL3

在EL0/EL1的系统中，MMU地址转换时，如果虚拟地址在0x00000000_ffffffff - 0x0000ffff_ffffffff范围，MMU会自动使用TTBR0_EL1指向的页表，进行地址转换；如果虚拟地址在0xffff0000_ffffffff - 0xffffffff_ffffffff范围，MMU会自动使用TTBR1_EL1指向的页表，进行地址转换

在EL2系统中，MMU地址转换时，会自动使用TTBR2_EL1指向的页表
在EL3系统中，MMU地址转换时，会自动使用TTBR3_EL1指向的页表

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013133317381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
#### 5、memory attributes介绍
translation tables为每一块region(entry)都定义了一个memory attributes条目,如同下面这个样子:
<font color=blue size=3>TODO:以linux kernel为例，在创建页表的时候，应该会设置这个memory attributes，有待看代码去验证</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013184314344.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
• Unprivileged eXecute Never (UXN) and Privileged eXecute Never (PXN) are execution
permissions.
• AF is the access flag.
• SH is the shareable attribute.
• AP is the access permission.
• NS is the security bit, but only at EL3 and Secure EL1.   <font color=red size=4>---- secure权限配置</font>
• Indx is the index into the MAIR_ELn

在这块region(entry)的memory attributes条目中的BIT4:2(index)指向了系统寄存器MAIR_ELn中的attr，MAIR_ELn共有8中attr选择
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013184453229.png#pic_center)
而每一个attr都有一种配置:
![在这里插入图片描述](https://img-blog.csdnimg.cn/202010131845126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
 <font color=blue size=3>有人可能会问，这里的inner和outter是什么意思呢，请参见前面的cache介绍的文章</font>

#### 6、memory tagging介绍

```c
When tagged addressing support is enabled, the top eight bits [63:56] of the virtual address are
ignored by the processor. It internally sets bit [55] to sign-extend the address to 64-bit format. The
top 8 bits can then be used to pass data. These bits are ignored for addressing and translation
faults. The TCR_EL1 has separate enable bits for EL0 and EL1
```

如果使用memory tagging, 虚拟地址的[63:56]用于传输签名数据，bit[55]表示是否需要签名.TCR_EL1也会有一个bit区分是给EL0用的还是给EL1用的

#### 7、启用hypervisor
(Two Stage Translations)
如果启用了hypervisor那么虚拟地址转换的过程将有VA--->PA变成了VA--->IPA--->PA, 也就是要经过两次转换.在guestos(如linux kernel)中转换的物理地址，其实不是真实的物理地址(假物理地址)，然后在EL2通过VTTBR0_EL2基地址的页表转换后的物理地址，才是真实的硬件地址

```c
IPA : intermediate physical address
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101410472724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
#### 8、Access permissions

```c
Access permissions are controlled through translation table entries. Access permissions control
whether a region is readable or writeable, or both, and can be set separately to EL0 for
unprivileged and access to EL1, EL2, and EL3 for privileged accesses, as shown in the following
table
```

>参考 ： SCTLR_EL1.WXN、SCTLR.UWXN

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014133906228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

执行权限:
- non-executable (Execute Never (XN))
- The Unprivileged Execute Never (UXN)
- Privileged Execute Never (PXN)

#### 9、MMU/cache相关的寄存器总结

MMU(address translation /TLB maintenance)、cache maintenance相关的寄存器

##### (1)、address translation
address translation 共计14个寄存器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013091804247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (2)、TLB maintenance
TLB maintenance数十个寄存器
(以下截取部分)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013091939225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

##### (3)、cache maintenance
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014093436827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014093518415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)




##### (4)、Base system registers 
系统寄存器中, 和MMU/Cache相关的寄存器有:
###### TTBR0_ELx TTBR1_ELx

(aarch64)
- TTBR0_EL1
- TTBR0_EL2
- TTBR0_EL3
- TTBR1_EL1
- VTTBR_EL2

(aarch32)
- TTBR0
- TTBR1
- HTTBR
- VTTBR


###### TCR_ELx

(aarch64)
- TCR_EL1
- TCR_EL2
- TCR_EL3
- VTCR_EL2

(aarch32)
- TTBCR(NS)
- HTCR
- TTBCR(S)
- VTCR

###### MAIR_ELx
- MAIR_EL1
- MAIR_EL2
- MAIR_EL3

#### 10、系统寄存器 --- TCR寄存器介绍
在ARM Core中(aarch64)，还有几个相关的系统寄存器:
- TCR_EL1   <font color=blue size=4>banked</font>
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



##### (1)、T1SZ、T0SZ
- T1SZ, bits [21:16] 通过TTBR1寻址的内存区域的大小偏移量，也就是TTBR1基地址下的一级页表的大小
- T0SZ, bits [5:0]

##### (2)、ORGN1、IRGN1、ORGN0、IRGN0
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013175241972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
其实可以总结为这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013175443123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (3)、SH1、SH0
SH1, bits [29:28]
SH0, bits [13:12]
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013184939895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
其实可以总结为这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013184959803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
Shareable的很容易理解，就是某个地址的可能被别人使用。我们在定义某个页属性的时候会给出。Non-Shareable就是只有自己使用。当然，定义成Non-Shareable不表示别人不可以用。某个地址A如果在核1上映射成Shareable，核2映射成Non-Shareable，并且两个核通过CCI400相连。那么核1在访问A的时候，总线会去监听核2，而核2访问A的时候，总线直接访问内存，不监听核1。显然这种做法是错误的。

对于Inner和Outer Shareable，有个简单的的理解，就是认为他们都是一个东西。在最近的ARM A系列处理器上上，配置处理器RTL的时候，会选择是不是把inner的传输送到ACE口上。当存在多个处理器簇或者需要双向一致性的GPU时，就需要设成送到ACE端口。这样，内部的操作，无论inner shareable还是outershareable，都会经由CCI广播到别的ACE口上。

##### (4)、TG0/TG1 - Granule size
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014112337111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (5)、IPS
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014121152796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (6)、EPD1、EPD0
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014121854445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (7)、TBI1、TBI0
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122151709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (8)、A1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122307975.png#pic_center)
##### (10)、AS
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122355798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

**除了以上介绍的bit之外，剩余的bit都是特有功能使用或reserved的**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201014122503871.png#pic_center)


#### 11、代码使用示例展

##### (1)、设置inner/outer cache的属性(只写模式/回写模式/write allocate/No-write allocate)

如下代码所示：

```c
#define TCR_IRGN_WBWA		((UL(1) << 8) | (UL(1) << 24))   //使用TTBR0和使用TTBR1时后的inner cache的属性设置

#define TCR_ORGN_WBWA		((UL(1) << 10) | (UL(1) << 26))   //使用TTBR0和使用TTBR1时后的outer cache的属性设置

#define TCR_CACHE_FLAGS	TCR_IRGN_WBWA | TCR_ORGN_WBWA   // inner + outer cache的属性值


ENTRY(__cpu_setup)
......
	/*
	 * Set/prepare TCR and TTBR. We use 512GB (39-bit) address range for
	 * both user and kernel.
	 */
	ldr	x10, =TCR_TxSZ(VA_BITS) | TCR_CACHE_FLAGS | TCR_SMP_FLAGS | \
			TCR_TG_FLAGS | TCR_ASID16 | TCR_TBI0 | TCR_A1
	tcr_set_idmap_t0sz	x10, x9

......
	msr	tcr_el1, x10
	ret					// return to head.S
ENDPROC(__cpu_setup)
```

**属性设置了1，也就是回写模式、write allocate模式**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015134024712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

### 二、optee系统中使用MMU
##### 1、Optee中的TTBR0/TTBR1

我们知道，在linux中将虚拟空间划分为了userspace和kernel space：
例如：
aarch64 : 
		0x0000_0000_0000_0000 - 0x0000_ffff_ffff_ffff是userspace
		0xffff_0000_0000_0000 - 0xffff_ffff_ffff_ffff是kernel space
aarch32 : 0-3G是userspace，3-4G是kernel space

当cpu发起读写内存时，cpu发起的是虚拟地址，如果是kernel地址，那么MMU将自动使用TTBR1做为页表基地址进行转换程物理地址，然后发送到AXI总线，完成真正物理地址的读写;
如果cpu发起的虚拟地址是userspace地址，那么MMU将使用TTBR0做为页表基地址进行转换程物理地址，然后发送到AXI总线，完成真正物理地址的读写;

那么在optee中是怎么样的呢？
在optee中，没用特别的将虚拟地址划分成kernel space和userspace，统一使用0-4G地址空间(无论是aarch32还是aarch64). 
在optee初始化时，禁用了TTBR1，所有整个optee的虚拟地址的专业，都是使用TTBR0做为基地址转换成物理地址，发送到AXI总线，完成真正的物理地址读写

```c
void core_init_mmu_regs(void)
{
	uint64_t mair;
	uint64_t tcr;
	paddr_t ttbr0;
	uint64_t ips = calc_physical_addr_size_bits(max_pa);

	ttbr0 = virt_to_phys(l1_xlation_table[0][get_core_pos()]);

	mair  = MAIR_ATTR_SET(ATTR_DEVICE, ATTR_DEVICE_INDEX);
	mair |= MAIR_ATTR_SET(ATTR_IWBWA_OWBWA_NTR, ATTR_IWBWA_OWBWA_NTR_INDEX);
	write_mair_el1(mair);

	tcr = TCR_RES1;
	tcr |= TCR_XRGNX_WBWA << TCR_IRGN0_SHIFT;
	tcr |= TCR_XRGNX_WBWA << TCR_ORGN0_SHIFT;
	tcr |= TCR_SHX_ISH << TCR_SH0_SHIFT;
	tcr |= ips << TCR_EL1_IPS_SHIFT;
	tcr |= 64 - __builtin_ctzl(CFG_LPAE_ADDR_SPACE_SIZE);

	/* Disable the use of TTBR1 */
	tcr |= TCR_EPD1;

	/*
	 * TCR.A1 = 0 => ASID is stored in TTBR0
	 * TCR.AS = 0 => Same ASID size as in Aarch32/ARMv7
	 */

	write_tcr_el1(tcr);
	write_ttbr0_el1(ttbr0);
	write_ttbr1_el1(0);
	}
```

##### 2、optee中的页表

optee在在内核中使用一个4GB的大页表，在user mode使用多个32M的小页表
注意下面的图来自官网，有点小问题，代码中没用使用到TTBR1，kernel和user都使用TTBR0
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916211040433.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

##### 3、tee内核页表基地址的配置

**将基地址写入到TTBR0**

在optee的start函数和cpu_on_handler函数中，都会调用core_init_mmu_regs()

```c
FUNC _start , :
......
	bl	core_init_mmu_regs

FUNC cpu_on_handler , :
......
	bl	core_init_mmu_regs
```

在core_init_mmu_regs中，获取当前cpu的页表基地址，写入到ttbr0寄存器中

```c
void core_init_mmu_regs(void)
{
	uint64_t mair;
	uint64_t tcr;
	paddr_t ttbr0;
	uint64_t ips = calc_physical_addr_size_bits(max_pa);

	ttbr0 = virt_to_phys(l1_xlation_table[0][get_core_pos()]);  //获取当前cpu的页表基地址

	mair  = MAIR_ATTR_SET(ATTR_DEVICE, ATTR_DEVICE_INDEX);
	mair |= MAIR_ATTR_SET(ATTR_IWBWA_OWBWA_NTR, ATTR_IWBWA_OWBWA_NTR_INDEX);
	write_mair_el1(mair);

	tcr = TCR_RES1;
	tcr |= TCR_XRGNX_WBWA << TCR_IRGN0_SHIFT;
	tcr |= TCR_XRGNX_WBWA << TCR_ORGN0_SHIFT;
	tcr |= TCR_SHX_ISH << TCR_SH0_SHIFT;
	tcr |= ips << TCR_EL1_IPS_SHIFT;
	tcr |= 64 - __builtin_ctzl(CFG_LPAE_ADDR_SPACE_SIZE);

	/* Disable the use of TTBR1 */
	tcr |= TCR_EPD1;

	/*
	 * TCR.A1 = 0 => ASID is stored in TTBR0
	 * TCR.AS = 0 => Same ASID size as in Aarch32/ARMv7
	 */

	write_tcr_el1(tcr);
	write_ttbr0_el1(ttbr0);   //将页表基地址写入到ttbr0寄存器
	write_ttbr1_el1(0);
}
```

##### 4、tee内核页表填充

在optee内核中，实现的是一个4G的大页表(一级页表, 32bit可表示4G空间)

在section段定义了一个三元数组，用于存放页表

```c
uint64_t l1_xlation_table[NUM_L1_TABLES][CFG_TEE_CORE_NB_CORE][NUM_L1_ENTRIES]
	__aligned(NUM_L1_ENTRIES * XLAT_ENTRY_SIZE) __section(".nozi.mmu.l1");
```

形态入下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916210910559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
**填充页表，那么要将哪些地址填进去呢？**
在core_init_mmu_map()函数中，调用core_init_mmu_tables创建页表，参数static_memory_map是一个数组，它指向系统中向optee已注册了的所有内存区域的地址

```c
void core_init_mmu_map(void)
{
......
	core_init_mmu_tables(static_memory_map);
......
}
```

```c
static struct tee_mmap_region static_memory_map[CFG_MMAP_REGIONS + 1];
```

```c
#define CFG_MMAP_REGIONS 13，//表示当前optee，最多支持注册13块内存区域
```

**调用core_init_mmu_tables()填充的页表**
参数mm就是刚才的static_memory_map，小于等于13块的内存区域， 填充方法如下

```c
void core_init_mmu_tables(struct tee_mmap_region *mm)
{
	uint64_t max_va = 0;
	size_t n;

#ifdef CFG_CORE_UNMAP_CORE_AT_EL0
	COMPILE_TIME_ASSERT(CORE_MMU_L1_TBL_OFFSET ==
			   sizeof(l1_xlation_table) / 2);
#endif
	max_pa = get_nsec_ddr_max_pa();

	for (n = 0; !core_mmap_is_end_of_table(mm + n); n++) {
		paddr_t pa_end;
		vaddr_t va_end;

		debug_print(" %010" PRIxVA " %010" PRIxPA " %10zx %x",
			    mm[n].va, mm[n].pa, mm[n].size, mm[n].attr);

		if (!IS_PAGE_ALIGNED(mm[n].pa) || !IS_PAGE_ALIGNED(mm[n].size))
			panic("unaligned region");

		pa_end = mm[n].pa + mm[n].size - 1;
		va_end = mm[n].va + mm[n].size - 1;
		if (pa_end > max_pa)
			max_pa = pa_end;
		if (va_end > max_va)
			max_va = va_end;
	}

	/* Clear table before use */
	memset(l1_xlation_table, 0, sizeof(l1_xlation_table));   ------------------------清空页表

	for (n = 0; !core_mmap_is_end_of_table(mm + n); n++)
		if (!core_mmu_is_dynamic_vaspace(mm + n))
			core_mmu_map_region(mm + n);   ------------------------填充当前cpu的页表

	/*
	 * Primary mapping table is ready at index `get_core_pos()`
	 * whose value may not be ZERO. Take this index as copy source.
	 */
	for (n = 0; n < CFG_TEE_CORE_NB_CORE; n++) {
		if (n == get_core_pos())
			continue;

		memcpy(l1_xlation_table[0][n],
		       l1_xlation_table[0][get_core_pos()],   ------------------------将当前cpu的页表，拷贝到其它cpu的页表中
		       XLAT_ENTRY_SIZE * NUM_L1_ENTRIES);
	}

	for (n = 1; n < NUM_L1_ENTRIES; n++) {
		if (!l1_xlation_table[0][0][n]) {
			user_va_idx = n;
			break;
		}
	}
	assert(user_va_idx != -1);

	COMPILE_TIME_ASSERT(CFG_LPAE_ADDR_SPACE_SIZE > 0);
	assert(max_va < CFG_LPAE_ADDR_SPACE_SIZE);
}
```

**在下列段代码中，循环遍历mm遍历，对于每个内存区域，做core_mmu_map_region运算**

```c
for (n = 0; !core_mmap_is_end_of_table(mm + n); n++)
		if (!core_mmu_is_dynamic_vaspace(mm + n))
			core_mmu_map_region(mm + n);
```


**core_mmu_map_region就是将每块内存区域，进行拆分，拆分程若干entry**

```c
void core_mmu_map_region(struct tee_mmap_region *mm)
{
	struct core_mmu_table_info tbl_info;
	unsigned int idx;
	vaddr_t vaddr = mm->va;
	paddr_t paddr = mm->pa;
	ssize_t size_left = mm->size;
	int level;
	bool table_found;
	uint32_t old_attr;

	assert(!((vaddr | paddr) & SMALL_PAGE_MASK));

	while (size_left > 0) {
		level = 1;

		while (true) {
			assert(level <= CORE_MMU_PGDIR_LEVEL);

			table_found = core_mmu_find_table(vaddr, level,
							  &tbl_info);
			if (!table_found)
				panic("can't find table for mapping");

			idx = core_mmu_va2idx(&tbl_info, vaddr);

			if (!can_map_at_level(paddr, vaddr, size_left,
					      1 << tbl_info.shift, mm)) {
				/*
				 * This part of the region can't be mapped at
				 * this level. Need to go deeper.
				 */
				if (!core_mmu_entry_to_finer_grained(&tbl_info,
					      idx, mm->attr & TEE_MATTR_SECURE))
					panic("Can't divide MMU entry");
				level++;
				continue;
			}

			/* We can map part of the region at current level */
			core_mmu_get_entry(&tbl_info, idx, NULL, &old_attr);
			if (old_attr)
				panic("Page is already mapped");

			core_mmu_set_entry(&tbl_info, idx, paddr, mm->attr);
			paddr += 1 << tbl_info.shift;
			vaddr += 1 << tbl_info.shift;
			size_left -= 1 << tbl_info.shift;

			break;
		}
	}
}
```

##### 5、virt_to_phys转换的过程的举例

**通过画图讲述了，在optee中，构建页表，然后完成一个virt_to_phys转换的过程.**

 - 1、系统注册了的若干块内存，划分为若干个大小为4K/8K的region，每个region的地址，写入到页表的entry中，这样就构建出了页表.
 - 2、将页表物理地址写入到TTBR0中
 - 3、开启MMU
 - 4、调用virt_to_phys时，使用MMU的AT S1E1R指令，将虚拟地址写入到MMU的Address translation中
 - 5、从par_el1中读出的物理地址，就是经过MMU转换后的

**注意:**
1、在optee中禁止了TTBR1，所以在optee的kernel mode中，也是使用TTBR0
2、optee中，只使用到了一级页表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918120413846.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)


------------------
><font color=blue size=4>**欢迎添加微信、微信群，多多交流**</font>
<img src="http://assets.processon.com/chart_image/604719347d9c082c92e419de.png">
# Cache Type Register(CTR)寄存器介绍



在ARMV8中，只有CTR_EL0，没有CTR_EL1/2/3

#### 1、CTR_EL0寄存器介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030132525347.png#pic_center)

##### (1)、DminLine/IminLine
>Log2 of the number of words in the smallest cache line of all the data caches and unified caches that are controlled by the PE.x

<font color=blue size=4>cache line的大小，cache_line_size = 4 * ($2^x$)， x=[19:16], [3:0]</font>

获取d-cache cache-line的示例代码:
```c
        .macro  dcache_line_size, reg, tmp
        mrs     \tmp, ctr_el0                   // read CTR
        ubfm    \tmp, \tmp, #16, #19            // cache line size encoding
        mov     \reg, #4                        // bytes per word
        lsl     \reg, \reg, \tmp                // actual cache line size
        .endm
````
其算法就是:
a. 读取ctr_el0比特16到19的数值 ： tmp = ctr_el0[19:16]
b. 计算cache line大小 : reg = (4 << tmp) = 4 * (2^tmp)

##### (2)、TminLine
>Tag minimum Line. Log2 of the number of words covered by Allocation Tags in the smallest cache
line of all caches which can contain Allocation tags that are controlled by the PE

<font color=blue size=4>cache line的大小，tag_size = 4 * ($2^x$)， x=[37:32]</font>


#### (3)、DIC
>Instruction cache invalidation requirements for data to instruction coherence
>
0b0 Instruction cache invalidation to the Point of Unification is required for data to instruction coherence.
0b1 Instruction cache invalidation to the Point of Unification is not required for data to instruction coherence.


##### (4)、IDC
>Data cache clean requirements for instruction to data coherence

0b0 Data cache clean to the Point of Unification is required for instruction to data coherence,
unless CLIDR_EL1.LoC == 0b000 or (CLIDR_EL1.LoUIS == 0b000 &&
CLIDR_EL1.LoUU == 0b000).
0b1 Data cache clean to the Point of Unification is not required for instruction to data
coherence.


##### (5)、CWG
Cache writeback granule. Log2 of the number of words of the maximum size of memory

##### （6）、ERG
Exclusives reservation granule. Log2 of the number of words of the maximum size of the reservation granule 

##### (7)、L1Ip
Level 1 instruction cache policy
>0b00 VMID aware Physical Index, Physical tag (VPIPT)
0b01 ASID-tagged Virtual Index, Virtual Tag (AIVIVT)
0b10 Virtual Index, Physical Tag (VIPT)
0b11 Physical Index, Physical Tag (PIPT)

#### 2、代码示例
在Linux Kernel代码中，读取CTR_EL0的地方只有三处，分别是：
- raw_icache_line_size
- raw_dcache_line_size
- read_ctr
```c
(**arch/arm64/include/asm/assembler.h**)
/*
 * raw_icache_line_size - get the minimum I-cache line size on this CPU
 * from the CTR register.
 */
	.macro	raw_icache_line_size, reg, tmp
	mrs	\tmp, ctr_el0			// read CTR
	and	\tmp, \tmp, #0xf		// cache line size encoding
	mov	\reg, #4			// bytes per word
	lsl	\reg, \reg, \tmp		// actual cache line size
	.endm

/*

/*
 * raw_dcache_line_size - get the minimum D-cache line size on this CPU
 * from the CTR register.
 */
	.macro	raw_dcache_line_size, reg, tmp
	mrs	\tmp, ctr_el0			// read CTR
	ubfm	\tmp, \tmp, #16, #19		// cache line size encoding
	mov	\reg, #4			// bytes per word
	lsl	\reg, \reg, \tmp		// actual cache line size
	.endm


	.macro	read_ctr, reg
alternative_if_not ARM64_MISMATCHED_CACHE_LINE_SIZE
	mrs	\reg, ctr_el0			// read CTR
	nop
alternative_else
	ldr_l	\reg, arm64_ftr_reg_ctrel0 + ARM64_FTR_SYSVAL
alternative_endif
	.endm
```
而raw_icache_line_size和raw_dcache_line_size并没有人直接调用；
read_ctr被 dcache_line_size 和 icache_line_size调用
```c
/*
 * dcache_line_size - get the safe D-cache line size across all CPUs
 */
	.macro	dcache_line_size, reg, tmp
	read_ctr	\tmp
	ubfm		\tmp, \tmp, #16, #19	// cache line size encoding
	mov		\reg, #4		// bytes per word
	lsl		\reg, \reg, \tmp	// actual cache line size
	.endm

/*
 * raw_icache_line_size - get the minimum I-cache line size on this CPU
 * from the CTR register.
 */
	.macro	raw_icache_line_size, reg, tmp
	mrs	\tmp, ctr_el0			// read CTR
	and	\tmp, \tmp, #0xf		// cache line size encoding
	mov	\reg, #4			// bytes per word
	lsl	\reg, \reg, \tmp		// actual cache line size
	.endm
```
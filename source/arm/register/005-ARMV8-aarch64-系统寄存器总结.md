## ARMV8-aarch64-系统寄存器总结

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016132108217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

- ELR_ELx  异常链接寄存器
该寄存器只有ELR_EL1 ELR_EL2 ELR_EL3, 没用ELR_EL0. 因为异常不会routing(target)到EL0.
例如在user mode时触发了一个target到EL1的irq异常，那么会将PC指针保持到ELR_EL1中，然后跳转到EL1的异常向量表中;
user mode时触发了一个target到EL3的irq异常，，那么会将PC指针保持到ELR_EL3中，然后跳转到EL3的异常向量表中;

- ELR_ELx  (exception Syndrome register )异常综合寄存器/异常状态寄存器 ： 反应异常的原因等信息
该寄存器只有ELR_EL1 ELR_EL2 ELR_EL3, 没用ELR_EL0.
例如:s 16bit指令的异常、32bit指令的异常、simd浮点运算的异常、MSR/MRS的异常....


- ELR_ELx  (Fault Address Register)  错误的地址寄存器
当取指令或取数据时，PC对齐错误或者watchpoint异常(PC alignment fault and Watchpoint exceptions)，会将错误的地址填入到该寄存器中;


- MAIR_EL1, (Memory Attribute Indirection Register) 内存属性寄存器
配置内存的属性，如Tagged Normal Memory、normal memory、device memory
如果是normal memory，那么inner和Outer的配置是Write-Through /Write-back/write Allocate/write non-Allocate等


- SCTLR_EL1, (System Control Register) 系统控制寄存器
如d-cache/i-cache/mmu的enable和disable

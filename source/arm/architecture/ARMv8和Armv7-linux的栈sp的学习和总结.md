# ARMv8/armv7/linux的栈/sp的学习和总结

#### 1、ARMV8 ARMV7的SP寄存器的介绍


##### (1)、ARMV7-aarch32的SP寄存器

在ARMV8-aarch32的状态下，有以下SP寄存器
- sp
- sp_usr
- sp_svc
- sp_abt
- sp_und
- sp_irq
- sp_fiq
- sp_mon
- sp_hyp

注意:在armv7上，arm有七种模式:user、system、supervisor、abort、undefined、irq、fiq， 再加两个扩展模式:hyp、monitor

##### (2)、ARMV8-aarch32的SP寄存器

ASRMV8为了与armv7兼容，在ARMV8-aarch32的状态下，SP寄存器同armv7的一致
- sp
- sp_usr
- sp_svc
- sp_abt
- sp_und
- sp_irq
- sp_fiq
- sp_mon
- sp_hyp

其实armv8的aarch32的这些寄存器，map到了Xx通用寄存器上:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201021102326647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)


##### (3)、ARMV8-aarch64的SP寄存器

在ARMV8-aarch64的状态下，有以下SP寄存器
- SP_EL0  //当PSTATE.SP=0(SPSel.SP == 0)，高的级别访问的sp就是sp_el0
- SP_EL1  //在EL1级别下使用
- SP_EL2  //在EL2级别下使用
- SP_EL3  //在EL3级别下使用

#### 2、ARMV8 ARMV7的SP寄存器的使用举例


##### (1)、aarch32状态下读写SP的示例
```c
(1)
FUNC thread_set_abt_sp , :
UNWIND(	.fnstart)
UNWIND(	.cantunwind)
	mrs	r1, cpsr
	cps	#CPSR_MODE_ABT     //-------------切换到abt模式
	mov	sp, r0             //-------------此时操作的sp就是sp_abt
	msr	cpsr, r1
	bx	lr
UNWIND(	.fnend)
END_FUNC thread_set_abt_sp

(2)
FUNC thread_set_und_sp , :
UNWIND(	.fnstart)
UNWIND(	.cantunwind)
	mrs	r1, cpsr
	cps	#CPSR_MODE_UND     //-------------切换到und模式
	mov	sp, r0             //-------------此时操作的sp就是sp_abt
	msr	cpsr, r1
	bx	lr
UNWIND(	.fnend)
END_FUNC thread_set_und_sp

(3)
FUNC thread_set_irq_sp , :
UNWIND(	.fnstart)
UNWIND(	.cantunwind)
	mrs	r1, cpsr
	cps	#CPSR_MODE_IRQ     //-------------切换到irq模式
	mov	sp, r0             //-------------此时操作的sp就是sp_abt
	msr	cpsr, r1
	bx	lr
UNWIND(	.fnend)
END_FUNC thread_set_irq_sp

(4)
FUNC thread_set_fiq_sp , :
UNWIND(	.fnstart)
UNWIND(	.cantunwind)
	mrs	r1, cpsr
	cps	#CPSR_MODE_FIQ     //-------------切换到fiq模式
	mov	sp, r0             //-------------此时操作的sp就是sp_abt
	msr	cpsr, r1
	bx	lr
UNWIND(	.fnend)
END_FUNC thread_set_fiq_sp

......

```

##### (2)、aarch64状态下读写SP的示例
```c

/* The handler of native interrupt. */
.macro native_intr_handler mode:req
.......                    // 进入中断异常后，默认的SP就是SP_EL1
	msr	spsel, #0          // 在这里，将SP切换成SP_EL0
	mov	sp, x1
.......
	adr	x16, thread_nintr_handler_ptr       // 真正的中断处理函数，使用的是SP_EL0
.......
	/* Switch back to SP_EL1 */
	msr	spsel, #1         // 退出中断之前，再将SP切换成SP_EL1

	/* Update core local flags */
	ldr	w0, [sp, #THREAD_CORE_LOCAL_FLAGS]
	lsr	w0, w0, #THREAD_CLF_SAVED_SHIFT
	str	w0, [sp, #THREAD_CORE_LOCAL_FLAGS]
.......
	eret
1:	b	eret_to_el0
.endm


```
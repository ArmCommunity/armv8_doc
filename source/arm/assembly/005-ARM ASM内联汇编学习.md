## ARM ASM内联汇编学习



### 格式

```c
__asm__ qualifiers ( 
    
    // 汇编代码部分
    
    : OutputOperands //在内联汇编代码中被修改的变量列表
    : InputOperands  //在内联汇编代码中用到的变量列表
    : Clobbers       //在内联汇编代码中用到的寄存器列表
);
```

qualifiers：一般是用 volatile 修饰词

（1）、OutputOperands //在内联汇编代码中被修改的变量列表
**[asmSymbolicName] "constraint"(cvariablename)**
asmSymbolicName：表示变量在内联汇编代码中的别名，代码中就可以通过%[asmSymbolicName]去使用该变量
cvariablename：表示变量原来的名字；
constraint:一般填=r


（2）、InputOperands  //在内联汇编代码中用到的变量列表
**[asmSymbolicName] "constraint"(cexpression)**
按OutputOperands列表的顺序再列一遍，但是constraint用数字代替从0开始，然后才是写其他只读变量，只读变量constraint填r

（3）、Clobbers       //在内联汇编代码中用到的寄存器列表
Clobbers: 一**般是"cc", "memory"开头**，然后接着填内联汇编中用到的通用寄存器和向量寄存器
"cc"表示内联汇编代码修改了标志寄存器；
"memory"表示汇编代码对输入和输出操作数执行内存读取或写入操作

### 示例1：

```c
const uint8_t *src = ...;
uint8_t *dst       = ...;
int neonLen        = ...;
const int test     = ...；
    
#ifdef __aarch64__  // armv8
    __asm__ volatile(
    
        // 汇编代码部分
      
        :[src]        "=r"(src),
         [dst]        "=r"(dst),
         [neonLen]    "=r"(neonLen)
        :[src]        "0"(src),
         [dst]        "1"(dst),
         [neonLen]    "2"(neonLen),
         [test]       "r"(test)
        :"cc", "memory", "v0", "v1", "v2", "v3", "v4", "v5",...
    );
#else   // armv7
    __asm__ volatile(
    
        // 汇编代码部分
    
        :[src]          "=r"(src),
         [dst]          "=r"(dst),
         [neonLen]      "=r"(neonLen)
        :[src]          "0"(src),
         [dst]          "1"(dst),
         [neonLen]      "2"(neonLen),
         [test]         "r"(test)
        :"cc", "memory", "q0", "q1", "q2", "q3", "q4", "q5",...
    );
#endif
```

### 示例2：

```c
static inline unsigned long arch_local_save_flags(void)
{
	unsigned long flags;
	asm volatile(
		"mrs	%0, daif		// arch_local_save_flags"
		: "=r" (flags)
		:
		: "memory");
	return flags;
}

/*
 * restore saved IRQ state
 */
static inline void arch_local_irq_restore(unsigned long flags)
{
	asm volatile(
		"msr	daif, %0		// arch_local_irq_restore"
	:
	: "r" (flags)
	: "memory");
}

static inline void arch_local_irq_disable(void)
{
	asm volatile(
		"msr	daifset, #2		// arch_local_irq_disable"
		:
		:
		: "memory");
}

#define local_fiq_enable()	asm("msr	daifclr, #1" : : : "memory")
#define local_fiq_disable()	asm("msr	daifset, #1" : : : "memory")
```

```c
#define sev()		asm volatile("sev" : : : "memory")
#define wfe()		asm volatile("wfe" : : : "memory")
#define wfi()		asm volatile("wfi" : : : "memory")

#define isb()		asm volatile("isb" : : : "memory")
#define dmb(opt)	asm volatile("dmb " #opt : : : "memory")
#define dsb(opt)	asm volatile("dsb " #opt : : : "memory")

#define mb()		dsb(sy)
#define rmb()		dsb(ld)
#define wmb()		dsb(st)
```

（这是自己写的virt_to_phys的代码，将虚拟地址写入寄存器，然后再从另外的寄存器读出物理地址）
```c
#define DEFINE_REG_READ_FUNC_(reg, type, asmreg)	\
static inline type read_##reg(void)			\
{							\
	type val;					\
							\
	asm volatile("mrs %0, " #asmreg : "=r" (val));	\
	return val;					\
}

#define DEFINE_U64_REG_READ_FUNC(reg) \
		DEFINE_REG_READ_FUNC_(reg, uint64_t, reg)

DEFINE_U64_REG_READ_FUNC(par_el1)

static inline void write_at_s1e1r(uint64_t va)
{
	asm volatile ("at	S1E1R, %0" : : "r" (va));
}
```

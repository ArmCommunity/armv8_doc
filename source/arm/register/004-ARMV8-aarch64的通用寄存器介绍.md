## ARMV8-aarch64的通用寄存器介绍


#### 1、通用寄存器
##### (1)、X0-X31
ARMv8有31个通用寄存器X0-X30, 还有SP、PC、XZR等寄存器
下面详细介绍写这些通用寄存器(general-purpose registers)：
- **X0-X7**  用于参数传递
- **X9-X15**  在子函数中使用这些寄存器时，直接使用即可, 无需save/restore. 在汇编代码中x9-x15出现的频率极低 
在子函数中使用这些寄存器时，直接使用即可, 无需save/restore. 在汇编代码中x9-x15出现的频率极低
- **X19-X29** 在callee子函数中使用这些寄存器时，需要先save这些寄存器，在退出子函数时再resotre
在callee子函数中使用这些寄存器时，需要先save这些寄存器，在退出子函数时再resotre
- **X8, X16-X18, X29, X30**  这些都是特殊用途的寄存器
-- X8： 用于返回结果
-- X16、X17 ：进程内临时寄存器
-- X18 ：resrved for ABI
-- X29 ：FP（frame pointer register）
-- X30 ：LR

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201022134326903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### (2)、特殊寄存器：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216132327635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
部分寄存器还可以当作32位的使用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216132422807.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
**sp(Stack pointer)**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216132523238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
EL1t: t表示选择SP_EL0
EL1h:h表示选择SP_ELx(x>0)

**PC(Program Counter)**
在armv7上PC是一个通用寄存器R15，在armv8上PC不在是一个寄存器，它不能直接被修改。必需使用一些隐式的指令来改变，如PC-relative load

**SPSR**
![\[外链图片转存失败,源站可能有防盗在这里插入!链机制,建描述\]议将图片上https://传(imbog-sdnimg.cn/l7ck2v0201216133126942.png)https://img-1blog.csdnimg.cn/20201216133126942.png)\]](https://img-blog.csdnimg.cn/20201216133354169.png)

>- N Negative result (N flag).
>- Z Zero result (Z) flag.
>- C Carry out (C flag).
>- V Overflow (V flag).
>- SS Software Step. Indicates whether software step was enabled when an exception was taken.
>- IL Illegal Execution State bit. Shows the value of PSTATE.IL immediately before the exception was taken.
>- D Debug mask：watchpoint,breakpoint
>- A SError (System Error) mask bit.
>- I IRQ mask bit.
>- F FIQ mask bit.
>- M[4] Execution state that the exception was taken from. A value of 0 indicates AArch64.
>- M[3:0] Mode or Exception level that an exception was taken from

 **PSTATE (Processor state)**
 在aarch64中，调用ERET从一个异常返回时，会将SPSR_ELn恢复到PSTATE中，将恢复：ALU的flag、执行状态(aarch64 or aarch32)、异常级别、processor branches
PSTATE.{N, Z, C, V}可以在EL0中被修改，其余的bit只能在ELx(x>0)中修改
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216133457790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
#### 2、aarch32  register
执行状态的切换(aarch64/aarch32)
>**当从aarch32切换到aarch64时：**
1、一些64位寄存器的高32bit，曾经被map到aarch32中的一些寄存器。这些高32bit是unknown状态
2、一些64位寄存器的高32bit，在aarch32中没有使用，这些高32bit还是以前的值;
3、如果是跳转到EL3，EL2是aarch32，那么ELR_EL2的高32bit是unknown
4、以下寄存器，不会再aarch32中使用到
— SP_EL0.
— SP_EL1.
— SP_EL2.
— ELR_EL1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216180323595.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

**aarch64到aarch32的map:**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216180558379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

**PSTATE at AArch32**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216180719362.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020121618073270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)




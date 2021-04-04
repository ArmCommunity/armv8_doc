## Generic Timer

#### 1、architecture

generic Timer由System Counter和per-core timer组成.
System Counter提供固定频率的counter，56-64bits宽度,1MHZ-50MHZ


![在这里插入图片描述](httpsimg-blog.csdnimg.cn20210125191528555.pngx-oss-process=imagewatermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

#### 2、processor中有哪些timer？
![在这里插入图片描述](httpsimg-blog.csdnimg.cn20210125192154491.pngx-oss-process=imagewatermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)


#### 3、寄存器介绍
##### （1）、Count and frequency

CNTPCT_EL0：reports the current system count value
CNTFRQ_EL0：reports the frequency of the system count

##### （2）、Timer registers
主要有三个寄存器：
![在这里插入图片描述](httpsimg-blog.csdnimg.cn20210125192235481.pngx-oss-process=imagewatermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
timer前缀是：
![在这里插入图片描述](httpsimg-blog.csdnimg.cn2021012519224776.pngx-oss-process=imagewatermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
##### （3）、Accessing the timers

• EL1 Physical and Virtual Timers
EL0 access to these timers is controlled by font color=blue size=4CNTKCTL_EL1font.
• EL2 Physical and Virtual Timers
When HCR_EL2.{TGE,E2H}=={1,1}, EL0 access to these timers is controlled by font color=blue size=4CNTKCTL_EL2：font. These timers were added as part of the support for the Armv8.1-A Virtualization Host Extension, which is beyond the scope of this
guide
• EL3 physical timer
S.EL1 and S.EL2 access to this timer is controlled by font color=blue size=4SCR_EL3.STfont.

#### 4、Configuring a timer
使用timer有两种方式：
- 使用CVAL, CVAL是64位的寄存器;
Timer Condition Met CVAL = System Count
- 使用TVAL, TVAL是32位的寄存器， TVAL随着system conter值的增加而减小.
CVAL = TVAL + System Counter
Timer Condition Met CVAL = System Count

#### 4、中断


timer产生的中断，只可以routing到当前core

timer中断的控制，在CTL寄存器中
• ENABLE – Enables the timer.
• IMASK – Interrupt mask. Enables or disables interrupt generation.
• ISTATUS – When ENABLE==1, reports whether the timer is firing (CVAL = System Count).
Must set ENABLE to 1 and clear IMASK to 0

The interrupt ID (INTID)由SBSA文档定义推荐的值， 注：Server Base System Architecture (SBSA)
![在这里插入图片描述](httpsimg-blog.csdnimg.cn2021012519532665.pngx-oss-process=imagewatermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

中断产生之后，会持续输出中断，直到如下条件发生
• IMASK is set to one, which masks the interrupt.
• ENABLE is cleared to 0, which disables the timer.
• TVAL or CVAL is written, so that firing condition is no longer met.

所以在产生一个timer中断后，软件需要在该中断变为deactivating之前，清除该中断。

#### 5、Timer virtualization

Virtual Count = Physical Count - offset
CNTVOFF_EL2配置offset，CNTVOFF_EL2只可以在EL3或EL2中访问

![在这里插入图片描述](httpsimg-blog.csdnimg.cn20210125195503341.pngx-oss-process=imagewatermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
如果EL2没有使用，那么offset为0,此时虚拟counter和物理conter值相等

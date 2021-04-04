# ARMV8 MMU内存管理中的Memory attributes和Cache policies

reserved



#### 1、MMU页表中的内存属性介绍

##### Memory attributes
在MMU translation tables中为每一个region（entry）定义了memory和cache属性
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201029093841243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
在该属性中的BIT[4:2]做为index指向指向了系统寄存器MAIR_ELn **（Cache policies**）, 系统寄存器MAIR_ELn分成8*8bytes
TLB做为一种特殊的cache，在它的entry中包含了memory type，所有修改MAIR_ELn寄存器后，在使用ISB指令或TLB invalidate操作之前，不会对TLB生效

##### MAIR_EL1
MAIR_EL1, Memory Attribute Indirection Register (EL1)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201029094137438.png#pic_center)

##### Cache policies
Attrx的定义(Cache policies)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201029094309428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
各个bit位的具体含义
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201029094452337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
例如optee中的内存属性配置如下:
```c
#define ATTR_DEVICE_INDEX		0x0
#define ATTR_IWBWA_OWBWA_NTR_INDEX	0x1
#define ATTR_INDEX_MASK			0x7

#define ATTR_DEVICE			(0x4)
#define ATTR_IWBWA_OWBWA_NTR		(0xff)

mair  = MAIR_ATTR_SET(ATTR_DEVICE, ATTR_DEVICE_INDEX);
mair |= MAIR_ATTR_SET(ATTR_IWBWA_OWBWA_NTR, ATTR_IWBWA_OWBWA_NTR_INDEX);
write_mair_el1(mair);
```
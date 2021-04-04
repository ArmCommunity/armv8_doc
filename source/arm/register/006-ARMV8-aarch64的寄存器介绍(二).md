## ARMV8-aarch64的寄存器介绍(二)

#### 1、aarch64通用寄存器
ARMV8-aarch64有31个64位的寄存器 ： x0-x30, 其中x29是Frame pointer（FP）, x30是procedure link register（LR）

#### 2、aarch64特殊寄存器
（sp pc spsr  elr xzr等）
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020102309411919.png#pic_center)
在aarch64中，没有x31或w31寄存器，但是在一些指令或软件编码中，经常将数字31做为XZR或SP

#### 4、Stack pointer(sp)寄存器介绍
默认情况下，来了一个异常后，选择当前异常级别的sp，例如来了一个异常到EL1, 那么将自动选择sp_el1做为sp;
但是呢，在高异常等级，通过修改spsel，也可以使用SP_EL0
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020102309485816.png#pic_center)
#### 4、PSTATE
PSTATE的bit位定义：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023095109326.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

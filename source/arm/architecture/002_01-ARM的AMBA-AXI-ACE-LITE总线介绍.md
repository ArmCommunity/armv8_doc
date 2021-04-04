## ARM AMBA/AXI/ACE/LITE总线介绍

#### 术语：
>Advanced Microcontroller Bus Architecture (AMBA) 
>Advanced System Bus (ASB)  
>Advanced Peripheral Bus (APB)
>Advanced High-performance Bus (AHB)
>Advanced Trace Bus (ATB)
>Advanced eXtensible Interface (AXI)
>AXI Coherency Extensions (ACE) 
>AMBA 5 Coherent Hub Interface (CHI)
>AHB-Lite
#### AMBA的发展
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222151721259.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
AHB- lite是AHB的子集。这个子集简化了带有单个主机的总线的设计

ACE扩展了AXI，引入了系统范围内的一致性。这种系统范围内的一致性允许多个处理器共享内存，并支持像big.LITTLE这样的技术。
ACE-lite协议支持单向一致性.
AXI-Stream协议设计用于从主服务器到从服务器的单向数据传输减少了信号路由，非常适合在fpga中实现。

2014年, AMBA5-CHI 重新设计高速运输层和功能的设计，以减少拥堵
2016年, AHB-Lite协议更新为AHB5，以补充Armv8-M架构
2019年,ATP是现有ATP的补充AMBA协议和被用于建模高层次内存访问行为在一个简洁，简单，和可移植的方法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222154338313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
一张复杂的使用框图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122215470873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

#### AXI channels
AXI规范描述了两个接口之间的点对点协议:一个主接口和一个从接口。该协议主要描述5个通道，框图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222154936865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
例如表示secure state的比特在AWPROT[1]和ARPROT[1]中.

####  Channel signals
##### Write channel signals
• Write Address
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222162656310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
• Write Data
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222162803550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

• Write Response
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122216375650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
##### Read channel signals
• Read Address
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222164715491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

• Read Data
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222164733440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

#### 功能的介绍
##### Protection level support （AWPROT and ARPROT）
AxPROT defines three levels of access protection

• AxPROT[0] (P) identifies an access as unprivileged or privileged:
o 1 indicates privileged access.
o 0 indicates unprivileged access

• AxPROT[1] (NS) identifies an access as Secure or Non-secure:
o 1 indicates a Non-secure transaction.
o 0 indicates a Secure transaction

• AxPROT[2] (I) indicates whether the transaction is an instruction access or a data access:
o 1 indicates an instruction access.
o 0 indicates a data access


##### Atomic accesses
有两种类型的原子访问：
- 锁(locked): 当一个Master正在访问slave，其它master再来访问该slave时将会被拒绝;
- 独占(exclusive)：当一个Master正在访问slave，其它master也可以访问该slave，但不能访问相同的memory范围

(1)、Locked accesses
locked主要用于过去的一些设备。AXI4不在支持locked了.AXI3必需支持locked。
在一个master去locked之前，需要确保没有其它master正在处理。带有AxLOCK信号集的事务表示被锁定的事务。一个被锁定的事务，将会拒绝master的访问.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222191023583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)


(2)、Exclusive accesses
独占访问可以由多个数据序列组成，但这所有事务都必须具有相同的地址空间.
硬件独占的访问监视器需要记录独占序列的事务信息，需要知道被访问的地址范围和Master的identity号.



# cache的一些基本概念介绍



#### 1、Cache的一些基本概念

###### (1)、cache的架构图(L1、L2、L3 cache)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023181355426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
###### (2)、cache的术语:set/way/line/index/tag/offset
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023181526280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

#### 2、Cache的一些属性概念

##### Cache分配策略(Cache allocation policy)
cache的分配策略是指我们什么情况下应该为数据分配cache line。cache分配策略分为读和写两种情况。

###### (1)、读分配(read allocation)
当CPU读数据时，发生cache缺失，这种情况下都会分配一个cache line缓存从主存读取的数据。默认情况下，cache都支持读分配。

###### (2)、写分配(write allocation)
当CPU写数据发生cache缺失时，才会考虑写分配策略。当我们不支持写分配的情况下，写指令只会更新主存数据，然后就结束了。当支持写分配的时候，我们首先从主存中加载数据到cache line中（相当于先做个读分配动作），然后会更新cache line中的数据。

##### Cache更新策略(Cache update policy)
cache更新策略是指当发生cache命中时，写操作应该如何更新数据。cache更新策略分成两种：写直通和回写。

###### (3)、写直通(write through)
当CPU执行store指令并在cache命中时，我们更新cache中的数据并且更新主存中的数据。cache和主存的数据始终保持一致。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023180118303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

###### (4)、写回(write back)
当CPU执行store指令并在cache命中时，我们只更新cache中的数据。并且每个cache line中会有一个bit位记录数据是否被修改过，称之为dirty bit（翻翻前面的图片，cache line旁边有一个D就是dirty bit）。我们会将dirty bit置位。主存中的数据只会在cache line被替换或者显示的clean操作时更新。因此，主存中的数据可能是未修改的数据，而修改的数据躺在cache中。cache和主存的数据可能不一致。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023180100458.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

##### properties of normal memory 
###### (5)、inner and outer
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023180033737.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
##### PoC 和 PoU

###### (6) Poc
Poc是值对于不同的Master看到的一致性的内存; 例如对于cores,DSP,DMA他们一致性的内存就是main memory，所以main memory是PoC这个点
>Point of Coherency (PoC). For a particular address, the PoC is the point at which all
observers, for example, cores, DSPs, or DMA engines, that can access memory, are
guaranteed to see the same copy of a memory location. Typically, this is the main external
system memory

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023180946120.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

###### (6) PoU
PoU是值指令和数据cache上一致的那个点，一般为L2 cache，如果系统中没有L2 cache，那么PoU为main memory
>Point of Unification (PoU). The PoU for a core is the point at which the instruction and
data caches and translation table walks of the core are guaranteed to see the same copy of
a memory location. For example, a unified level 2 cache would be the point of unification
in a system with Harvard level 1 caches and a TLB for caching translation table entries.
If no external cache is present, main memory would be the Point of Unification

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023181002770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
#### 3、cache的使用示例
###### (1)、一个两路组相连的cache
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023181836251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
###### (2)、一个32kb 四路组相连的cache
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201023182039627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
###### (3)、Inclusive 和 exclusive cache
wwww
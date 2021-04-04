# ARMV8的cache的指令集介绍



Armv8里定义的Cache的管理的操作有三种：

- **无效**（Invalidate） 整个高速缓存或者某个高速缓存行。高速缓存上的数据会被丢弃。
- **清除**（Clean） 整个高速缓存或者某个高速缓存行。相应的高速缓存行会被标记为脏，数据会写回到下一级高速缓存中或者主存储器中。
- **清零**（Zero） 在某些情况下，对高速缓存进行清零操作起到一个预取和加速的功效，比如当程序需要使用一大块临时内存，在初始化阶段对这个内存进行清零操作，这时高速缓存控制器会主动把这些零数据写入高速缓存行中。若程序主动使用高速缓存的清零操作，那么将大大减少系统内部总线的带宽。

对高速缓存的操作可以指定不同的范围。
- 整块高速缓存。
- 某个虚拟地址。
- 特定的高速缓存行或者组和路。

另外在ARMv8架构中最多可以支持7级的高速缓存，L1～L7高速缓存。当对一个高速缓存行进行操作时，我们需要知道高速缓存操作的范围。ARMv8架构中从处理器到所有内存的角度分成如下几个视角。
- PoC（Point of Coherency，全局缓存一致性角度）
- PoU（Point of  Unification，处理器缓存一致性角度


有了上面的这些概念，然后我们再看下cache的指令格式:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118154912892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
function中的G和D一般和其它function组合使用的，例如
**DC IGDVAC** ：操作d-cache，根据虚拟地址无效掉data和Allocation Tags，操作范围到PoC
>Invalidate data and Allocation Tags in data cache by address to Point of Coherency.
>
**DC IGVAC** ：操作d-cache，根据虚拟地址无效掉Allocation Tags，操作范围到PoC

>Invalidate Allocation Tags in data cache by address to Point of Coherency

最后，我们再看armv8文档中定义的cache指令集：

```c
DC CISW, Data or unified Cache line Clean and Invalidate by Set/Way
DC CSW, Data or unified Cache line Clean by Set/Way
DC CVAU, Data or unified Cache line Clean by VA to PoU
DC ZVA, Data Cache Zero by VA

IC IALLU, Instruction Cache Invalidate All to PoU
IC IALLU{, <Xt>}
IC IALLUIS, Instruction Cache Invalidate All to PoU, Inner Shareable
IC IALLUIS{, <Xt>}
IC IVAU, Instruction Cache line Invalidate by VA to PoU
IC IVAU{, <Xt>}

DC CIVAC, Data or unified Cache line Clean and Invalidate by VA to PoC
DC CVAC, Data or unified Cache line Clean by VA to PoC
DC CVAP, Data or unified Cache line Clean by VA to PoP
DC GVA, Data Cache set Allocation Tag by VA
DC GZVA, Data Cache set Allocation Tags and Zero by VA
DC IGDSW, Data, Allocation Tag or unified Cache line Invalidate of Data and Allocation Tags by Set/Way
DC IGDVAC, Data, Allocation Tag or unified Cache line Invalidate of Allocation Tags by VA to PoC
DC IGSW, Data, Allocation Tag or unified Cache line Invalidate of Allocation Tags by Set/Way
DC IGVAC, Data, Allocation Tag or unified Cache line Invalidate of Allocation Tags by VA to PoC
DC ISW, Data or unified Cache line Invalidate by Set/Way
DC IVAC, Data or unified Cache line Invalidate by VA to PoC
DC CVADP, Data or unified Cache line Clean by VA to PoDP

ARMv8.5-MemTag:
DC CGDSW, Data, Allocation Tag or unified Cache line Clean of Data and Allocation Tags by Set/Way
DC CGDVAC, Data, Allocation Tag or unified Cache line Clean of Allocation Tags by VA to PoC
DC CGDVADP, Data, Allocation Tag or unified Cache line Clean of Allocation Tags by VA to PoDP
DC CGDVAP, Data, Allocation Tag or unified Cache line Clean of Data and Allocation Tags by VA to PoP
DC CGSW, Data, Allocation Tag or unified Cache line Clean of Allocation Tags by Set/Way
DC CGVAC, Data, Allocation Tag or unified Cache line Clean of Allocation Tags by VA to PoC
DC CGVADP, Data, Allocation Tag or unified Cache line Clean of Data and Allocation Tags by VA to PoDP
DC CGVAP, Data, Allocation Tag or unified Cache line Clean of Allocation Tags by VA to PoP
DC CIGDSW, Data, Allocation Tag or unified Cache line Clean and Invalidate of Data and Allocation Tags by Set/Way
DC CIGDVAC, Data, Allocation Tag or unified Cache line Clean and Invalidate of Data and Allocation Tags by VA to PoC
DC CIGSW, Data, Allocation Tag or unified Cache line Clean and Invalidate of Allocation Tags by Set/Way
DC CIGVAC, Data, Allocation Tag or unified Cache line Clean and Invalidate of Allocation Tags by VA to PoC


```

cache操作的使用示例:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118205433501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)


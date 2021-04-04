# cache在linux和optee中的应用



在trustzone技术背景下，linux+optee系统环境，当cpu访问一块内存时，会经过memory filter的过滤，如果我将一块内存配置成secure memory，那么当non-secure cpu发起访问的时候，会被memroy filter挡回去，返回一个AXI Error.
但是在secure memory已经缓存到了cache的情况下，如果non-secure cpu再去访问该内存，cpu直接读写cache，就不经过memory filter，那么是如何保证cache安全的呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030093854794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
第一种情况: 
在配置该内存位secure属性后，在optee中建立MMU页表的时候，会置上页表entry中的的安全属性(NS=0).
那么当cpu发起读写访问的时候，经过MMU转换成物理地址，然后再去读写cache或main memory;

此时linux中也建立页表，在linux中在页表中故意将该地址范围配置成non-secure属性。
那么在linux中读取该地址数据时，发现hit了，不就拿到该地址段数据了吗?
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030093936539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

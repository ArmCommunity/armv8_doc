## ARMV8的exclusive和inexclusive的介绍

AMRv8-aarch64架构中的A64指令中，提供了如下的一些exclusive指令，用来支持exclusive操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929140633783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)

那为什么，arm在加入exclusive指令呢？加入这个，主要是为了解决多核情况下，锁的竞争问题。


在软件层面，对于共享资源的访问，会设定一个锁，只有能拿到这个锁的程序，才能够访问共享资源，而没有拿到锁的程序，就不能访问该共享资源。

```c
static int g_lock;

void lock()
{
	while(!g_lock);
	g_lock = 1;  // 上锁
}

void unlock()
{
	g_lock = 0;  //释放锁
}
```

在使用操作系统后，这个程序也可能会出现潜在的问题，因为操作系统会调度应用程序。假设应用程序A在读取到锁状态后，值为0，表示当前锁没有被其他程序占有，正当自己要将锁锁上的时候，操作系统进行了调度，应用程序B执行。程序B也读取锁状态，发现值为0，表示当前锁没有被其他程序占有，然后将锁给锁上，访问共享资源。此时，操作系统又进行了调度，应用程序A执行，将锁给锁上，然后访问共享资源。那此时，就出现了2个应用程序，同时访问共享资源，锁的作用，被屏蔽了。


(单核的情况下)为了解决这个问题，可以在获取锁的时候，屏蔽中断，那这个时候，操作系统就不能调度，也就避免上面出现的问题

```c
static int g_lock;

void lock()
{
	mask_interrupt();
	while(!g_lock);
	g_lock = 1;  // 上锁
	unmask_interrupt();
}

void unlock()
{
	g_lock = 0;  //释放锁
}
```

但是在多核的情况下，各个cpu是并行执行的，因此这个时候，还会出现，上面描述的2个程序，同时获取到锁的状态，而这个时候，通过禁止中断，是没有效果的.
为了解决多核情况下的锁竞争问题，arm引入了exclusive操作，并添加了相应的指令
exclusive的操作的核心，就是会将锁，用一个状态机进行维护，该状态机有2种状态，open状态和exclusive状态。要想成功的对锁进行上锁，状态必须要从exclusive状态切换到open状态，其他状态，都是失败的。

为了支持exclusive操作，在A64，新增了LDXR和STXR指令。



在A32和T32下，也加入LDREX和STREX指令来支持
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929141935689.png)


LDXR指令，将状态从open状态切换到exclusive状态，STXR指令，将状态从exclusive状态切换到open状态，这个就表示store exclusive操作成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200929141440473.png#pic_center)



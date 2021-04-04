## A64的load/store指令总结



#### 1、Load-Store Single Register 单寄存器读写

```c
ldr{<size>} Rd, <addr>
str{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 2、Load-Store Single Register (unscaled offset) offset为-256 ~ +256对齐读写

```c
ldur{<size>} Rd, <addr>
stur{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 3、Load-Store Pair  双寄存器读写

```c
ldp{<size>} Rd, <addr>
stp{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 4、Load-Store Non-temporal Pair 直接读写外存，跳过cache

```c
ldnp{<size>} Rd, <addr>
stnp{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 5、Load-Store Unprivileged 以EL0身份读写

```c
ldtr{<size>} Rd, <addr>
sttr{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 6、Load-Store Exclusive 独占

```c
ldxr{<size>} Rd, <addr>
stxr{<size>} Rd, <addr>

ldxp{<size>} Rd, <addr>
stxp{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 7、Load-Acquire / Store-Release  带有aruire/release语义的读写

```c
（Non-exclusive）
ldar{<size>} Rd, <addr>
stlr{<size>} Rd, <addr>

（exclusive）
ldaxr{<size>} Rd, <addr>
stlxr{<size>} Rd, <addr>

<size> : b h  sb  sh sw
```

#### 8、总结以上指令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200720170636577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)







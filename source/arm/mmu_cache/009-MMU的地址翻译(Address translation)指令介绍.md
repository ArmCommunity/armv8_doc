# MMU的地址翻译(Address translation)指令介绍



Address translation system instructions

**AT指令的语法格式：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111915071585.png#pic_center)
**有了上面的语法格式后，就非常好理解armv8的MMU提供了14条AT指令了：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119150939390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
MMU的地址翻译一般都是自动进行的，在当前的linux kernel(kernel-4.14)中还真找不到使用AT指令的代码。而在optee中是可以找到一个示例的，如下：

```c

static bool arm_va2pa_helper(void *va, paddr_t *pa)
{
        uint32_t exceptions = thread_mask_exceptions(THREAD_EXCP_ALL);
        paddr_t par;
        paddr_t par_pa_mask;
        bool ret = false;

#ifdef ARM32
        write_ats1cpr((vaddr_t)va);
        isb();
#ifdef CFG_WITH_LPAE
        par = read_par64();
        par_pa_mask = PAR64_PA_MASK;
#else
        par = read_par32();
        par_pa_mask = PAR32_PA_MASK;
#endif
#endif /*ARM32*/

#ifdef ARM64
        write_at_s1e1r((vaddr_t)va);
        isb();
        par = read_par_el1();
        par_pa_mask = PAR_PA_MASK;
#endif
        if (par & PAR_F)
                goto out;
        *pa = (par & (par_pa_mask << PAR_PA_SHIFT)) |
                ((vaddr_t)va & ((1 << PAR_PA_SHIFT) - 1));

        ret = true;
out:
        thread_unmask_exceptions(exceptions);
        return ret;
}

```

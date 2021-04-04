## ARMV8-aarch64的通用寄存器和系统寄存器介绍


### 一、通用寄存器

#### 1、X0-X31
ARMv8有31个通用寄存器X0-X30, 还有SP、PC、XZR等寄存器
下面详细介绍写这些通用寄存器(general-purpose registers)：
- **X0-X7**  用于参数传递
- **X9-X15**  在子函数中使用这些寄存器时，直接使用即可, 无需save/restore. 在汇编代码中x9-x15出现的频率极低 
在子函数中使用这些寄存器时，直接使用即可, 无需save/restore. 在汇编代码中x9-x15出现的频率极低
- **X19-X29** 在callee子函数中使用这些寄存器时，需要先save这些寄存器，在退出子函数时再resotre
在callee子函数中使用这些寄存器时，需要先save这些寄存器，在退出子函数时再resotre
- **X8, X16-X18, X29, X30**  这些都是特殊用途的寄存器
-- X8： 用于返回结果
-- X16、X17 ：进程内临时寄存器
-- X18 ：resrved for ABI
-- X29 ：FP（frame pointer register）
-- X30 ：LR

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201022134326903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70#pic_center)
#### 2、特殊寄存器：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216132327635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
部分寄存器还可以当作32位的使用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216132422807.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
**sp(Stack pointer)**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216132523238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
EL1t: t表示选择SP_EL0
EL1h:h表示选择SP_ELx(x>0)

**PC(Program Counter)**
在armv7上PC是一个通用寄存器R15，在armv8上PC不在是一个寄存器，它不能直接被修改。必需使用一些隐式的指令来改变，如PC-relative load

**SPSR**
![\[外链图片转存失败,源站可能有防盗在这里插入!链机制,建描述\]议将图片上https://传(imbog-sdnimg.cn/l7ck2v0201216133126942.png)https://img-1blog.csdnimg.cn/20201216133126942.png)\]](https://img-blog.csdnimg.cn/20201216133354169.png)

>- N Negative result (N flag).
>- Z Zero result (Z) flag.
>- C Carry out (C flag).
>- V Overflow (V flag).
>- SS Software Step. Indicates whether software step was enabled when an exception was taken.
>- IL Illegal Execution State bit. Shows the value of PSTATE.IL immediately before the exception was taken.
>- D Debug mask：watchpoint,breakpoint
>- A SError (System Error) mask bit.
>- I IRQ mask bit.
>- F FIQ mask bit.
>- M[4] Execution state that the exception was taken from. A value of 0 indicates AArch64.
>- M[3:0] Mode or Exception level that an exception was taken from

 **PSTATE (Processor state)**
 在aarch64中，调用ERET从一个异常返回时，会将SPSR_ELn恢复到PSTATE中，将恢复：ALU的flag、执行状态(aarch64 or aarch32)、异常级别、processor branches
PSTATE.{N, Z, C, V}可以在EL0中被修改，其余的bit只能在ELx(x>0)中修改
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216133457790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
#### 3、aarch32  register
执行状态的切换(aarch64/aarch32)
>**当从aarch32切换到aarch64时：**
1、一些64位寄存器的高32bit，曾经被map到aarch32中的一些寄存器。这些高32bit是unknown状态
2、一些64位寄存器的高32bit，在aarch32中没有使用，这些高32bit还是以前的值;
3、如果是跳转到EL3，EL2是aarch32，那么ELR_EL2的高32bit是unknown
4、以下寄存器，不会再aarch32中使用到
— SP_EL0.
— SP_EL1.
— SP_EL2.
— ELR_EL1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216180323595.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

**aarch64到aarch32的map:**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216180558379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

**PSTATE at AArch32**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216180719362.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020121618073270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)


### 二、System registers


>**AArch64 registers：**
>• Special-purpose registers.
• Base system registers 
• VMSA-specific registers
• ID registers on page
• Performance monitors registers
• Debug registers
• RAS registers
• Generic timer registers
• Cache maintenance system instructions
• Address translation system instructions
• TLB maintenance system instructions
• Prediction restriction System instructions


#### 1、Special-purpose registers
3个ELR， 4个SP， 8个SPSR
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213160601973.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
#### 2、Base system registers 
118个base system registers
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213165213122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)

#### 3、VMSA-specific registers
24个VMSA寄存器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213170130651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEzNTA4Nw==,size_16,color_FFFFFF,t_70)
#### 4、Address translation system instructions
14个地址翻译指令
>AT S12E0R
AT S12E0W
AT S12E1R
AT S12E1W
AT S1E0R
AT S1E0W
AT S1E1R
AT S1E1RP
AT S1E1W
AT S1E1WP
AT S1E2R
AT S1E2W
AT S1E3R
AT S1E3W

#### 5、TLB maintenance system instructions
78个TLB一致性指令
>TLBI ALLE1
TLBI ALLE1IS
TLBI ALLE1OS
TLBI ALLE2
TLBI ALLE2IS
TLBI ALLE2OS
TLBI ALLE3
TLBI ALLE3IS
TLBI ALLE3OS
TLBI ASIDE1 
TLBI ASIDE1IS
TLBI ASIDE1OS
TLBI IPAS2E1 
TLBI IPAS2E1IS
TLBI IPAS2E1OS
TLBI IPAS2LE1 
TLBI IPAS2LE1IS
TLBI IPAS2LE1OS
TLBI RIPAS2E1
TLBI RIPAS2E1IS
TLBI RIPAS2E1OS
TLBI RIPAS2LE1 
TLBI RIPAS2LE1IS
TLBI RIPAS2LE1OS
TLBI RVAAE1
TLBI RVAAE1IS
TLBI RVAAE1OS
TLBI RVAALE1 
TLBI RVAALE1IS
TLBI RVAALE1OS
TLBI RVAE1
TLBI RVAE1IS
TLBI RVAE1OS
TLBI RVAE2
TLBI RVAE2IS
TLBI RVAE2OS
TLBI RVAE3
TLBI RVAE3IS
TLBI RVAE3OS
TLBI RVALE1 
TLBI RVALE1IS
TLBI RVALE1OS
TLBI RVALE2
TLBI RVALE2IS
TLBI RVALE2OS
TLBI RVALE3
TLBI RVALE3IS
TLBI RVALE3OS
TLBI VAAE1
TLBI VAAE1IS
TLBI VAAE1OS
TLBI VAALE1 
TLBI VAALE1IS
TLBI VAALE1OS
TLBI VAE1
TLBI VAE1IS
TLBI VAE1OS
TLBI VAE2
TLBI VAE2IS
TLBI VAE2OS
TLBI VAE3
TLBI VAE3IS
TLBI VAE3OS
TLBI VALE1
TLBI VALE1IS
TLBI VALE1OS
TLBI VALE2
TLBI VALE2IS
TLBI VALE2OS
TLBI VALE3
TLBI VALE3IS
TLBI VALE3OS
TLBI VMALLE1
TLBI VMALLE1IS
TLBI VMALLE1OS
TLBI VMALLS12E1
TLBI VMALLS12E1IS
TLBI VMALLS12E1OS
#### 6、Cache maintenance system instructions
31个cache一致性指令
>DC CGDSW
DC CGDVAC
DC CGDVADP
DC CGDVAP
DC CGSW
DC CGVAC
DC CGVADP
DC CGVAP
DC CIGDSW
DC CIGDVAC
DC CIGSW
DC CIGVAC
DC CISW
DC CIVAC
DC CSW
DC CVAC
DC CVADP
DC CVAP
DC CVAU
DC GVA
DC GZVA
DC IGDSW
DC IGDVAC
DC IGSW
DC IGVAC
DC ISW
DC IVAC
DC ZVA
IC IALLU
IC IALLUIS
IC IVAU

#### 7、Generic timer registers
22个generic Timer寄存器
>CNTFRQ_EL0
CNTHV_CTL_EL2
CNTHV_CVAL_EL2
CNTHV_TVAL_EL2
CNTHVS_CTL_EL2
CNTHVS_CVAL_EL2
CNTHVS_TVAL_EL2
CNTKCTL_EL1
CNTP_CTL_EL0
CNTP_CVAL_EL0
CNTP_TVAL_EL0
CNTPCT_EL0
CNTPCTSS_EL0
CNTPOFF_EL2
CNTPS_CTL_EL1
CNTPS_CVAL_EL1
CNTPS_TVAL_EL1
CNTV_CTL_EL0
CNTV_CVAL_EL0
CNTV_TVAL_EL0
CNTVCT_EL0
CNTVCTSS_EL0

#### 8、RAS registers
16个RAS寄存器
>DISR_EL1
ERRIDR_EL1
ERRSELR_EL1
ERXADDR_EL1
ERXCTLR_EL1
ERXFR_EL1
ERXMISC0_EL1
ERXMISC1_EL1
ERXMISC2_EL1
ERXMISC3_EL1
ERXPFGCDN_EL1
ERXPFGCTL_EL1
ERXPFGF_EL1
ERXSTATUS_EL1
VDISR_EL2
VSESR_EL2
#### 9、Debug registers
23个debug寄存器
>DBGAUTHSTATUS_EL1
DBGBCR<n>_EL1
DBGBVR<n>_EL1
DBGCLAIMCLR_EL1
DBGCLAIMSET_EL1
DBGDTR_EL0
DBGDTRRX_EL0
DBGDTRTX_EL0
DBGPRCR_EL1
DBGVCR32_EL2
DBGWCR<n>_EL1
DBGWVR<n>_EL1
DLR_EL0
DSPSR_EL0
MDCCINT_EL1
MDCCSR_EL0
MDRAR_EL1
MDSCR_EL1
OSDLR_EL1
OSDTRRX_EL1
OSDTRTX_EL1
OSECCR_EL1
OSLAR_EL1
#### 10、Performance monitors registers
19个performance寄存器
>PMCCFILTR_EL0
PMCCNTR_EL0
PMCEID0_EL0
PMCEID1_EL0
PMCNTENCLR_EL0
PMCNTENSET_EL0
PMCR_EL0
PMEVCNTR<n>_EL0
PMEVTYPER<n>_EL0
PMINTENCLR_EL1
PMINTENSET_EL1
PMMIR_EL1
PMOVSCLR_EL0
PMOVSSET_EL0
PMSELR_EL0
PMSWINC_EL0
PMUSERENR_EL0
PMXEVCNTR_EL0
PMXEVTYPER_EL0
#### 11、ID registers
40个ID寄存器
>CCSIDR2_EL1
CCSIDR_EL1
CLIDR_EL1
CSSELR_EL1
CTR_EL0
DCZID_EL0
GMID_EL1
ID_AA64AFR0_EL1
ID_AA64AFR1_EL1
ID_AA64DFR0_EL1
ID_AA64DFR1_EL1
ID_AA64ISAR0_EL1
ID_AA64ISAR1_EL1
ID_AA64MMFR0_EL1
ID_AA64MMFR1_EL1
ID_AA64MMFR2_EL1
ID_AA64PFR0_EL1
ID_AA64PFR1_EL1
ID_AFR0_EL1 ID_AFR0_EL1
ID_DFR0_EL1 ID_DFR0_EL1
ID_DFR1_EL1 ID_DFR1_EL1
ID_ISAR0_EL1
ID_ISAR1_EL1
ID_ISAR2_EL1
ID_ISAR3_EL1
ID_ISAR4_EL1
ID_ISAR5_EL1
ID_ISAR6_EL1
ID_MMFR0_EL1
ID_MMFR1_EL1
ID_MMFR2_EL1
ID_MMFR3_EL1
ID_MMFR4_EL1
ID_MMFR5_EL1
ID_PFR0_EL1
ID_PFR1_EL1
ID_PFR2_EL1
MIDR_EL1
MPIDR_EL1
REVIDR_EL1

#### 12、Prediction restriction System instructions
3个预测限制寄存器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213171118973.png)

#### 13、必备寄存器总结

><font color=blue size=4>PSTATE：</font>程序状态寄存器
><br>
<font color=blue size=4>SP_ELx(x>0)：</font>在EL1/EL2/EL3 level中，如果spsel=0，则使用SP_ELx(x>0)
<font color=blue size=4>SP_EL0 ：</font>在所有level中，如果spsel=1, 则使用SP_EL0
><br>
<font color=blue size=4>SPSR ：</font>save program status register 备份的程序状态寄存器
><br>
<font color=blue size=4>ELR_ELx(x>0)：</font>异常链接寄存器，记录着异常时程序的返回地址
><br>
<font color=blue size=4>ESR_ELx(x>0) ：</font>同步异常, 异常特征寄存器
<font color=blue size=4>FAR_ELx(x>0)：</font> 同步异常, 异常时的错误地址
><br>
<font color=blue size=4>VBAR_ELx(x>0)：</font> 向量表基地址寄存器
><br>
<font color=blue size=4>TTBRn_ELx(n=1,2、x>0)：</font> 地址翻译基地址寄存器
<font color=blue size=4>MAIR_ELx(x>0) ：</font>内存属性寄存器
><br>
<font color=blue size=4>PAR_EL1：</font> 物理地址寄存器, 当使用指令操作MMU进行VA到PA的转换时，物理地址由PAR_EL1输出.
><br>
<font color=blue size=4>SCR_EL3 ：</font>安全配置寄存器
<font color=blue size=4>SCTLR_ELx(x>0）：</font> 系统控制寄存器
<font color=blue size=4>TCR_ELx(x>0)：</font> 地址翻译控制寄存器


------------------
><font color=blue size=4>**欢迎添加微信、微信群，多多交流**</font>
<img src="http://assets.processon.com/chart_image/604719347d9c082c92e419de.png">

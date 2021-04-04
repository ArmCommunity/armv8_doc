## ARM Core的分类和总结


### 1、A77
#### (1)、基本信息
- a high-performance and low-power Arm product
- armv8.5
- CPU interface : gicv4
#### (2)、cache/TLB
>L1 d-TLB 48-entry fully associative，支持4KB, 16KB, 64KB, 2MB, and 512MB的页面
L1 i-TLB 48-entry fully associative，支持4KB, 16KB, 64KB, 2MB, and 32MB的页面
L2 TLB  5-way set associative,1280-entry,支持4KB, 16KB, 64KB, 2MB, 32MB,512MB, and 1GB的BLOCK size
>.
>L1 d-cache 64KB, 4-way set associative, 64-byte cache lines
L1 i-ache  64KB, 4-way set associative, 64-byte cache lines
L2 cache  8-way set associative, 可选128KB, 256KB, or 512KB

### 1、A76
#### (1)、基本信息
- a high-performance and low-power Arm product
- armv8.5
- CPU interface : gicv4

#### (2)、cache/TLB
>L1 d-TLB 48-entry fully associative，支持4KB, 16KB, 64KB, 2MB, and 512MB的页面
L1 i-TLB 48-entry fully associative，支持4KB, 16KB, 64KB, 2MB, and 32MB的页面
L2 TLB  5-way set associative,1280-entry,支持4KB, 16KB, 64KB, 2MB, 32MB,512MB, and 1GB的BLOCK size
>.
>L1 d-cache 64KB, 4-way set associative, 64-byte cache lines
L1 i-ache  64KB, 4-way set associative, 64-byte cache lines
L2 cache  8-way set associative, 可选128KB, 256KB, or 512KB

### 3、A53

#### (1)、基本信息


- a mid-range, low-power processor
- armv8
- CPU interface : gicv4
- 上市时间：2013-2014

#### (2)、cache/TLB
>Micro TLB : instruction and data TLB 各有10个entries，全相连
Main TLB : 512-entry, 4-way set-associative, 不支持1GB的block size，如果访问1GB,则会拆分成两个512M执行
>
>L1 d-cache  4-way set associative, 64-byte cache lines, 可选8K/16K/32K/64K
L1 i-ache  2-way set associative, 64-byte cache lines, 可选8K/16K/32K/64K
L2 cache  16-way set associative,64-byte cache lines, 可选128KB, 256KB, 512KB, 1MB and 2MB

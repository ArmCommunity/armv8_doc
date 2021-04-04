## ARM ToolChains介绍


#### 1、toolchains版本的介绍
在linaro官网有众多toolchains的版本，目前比较常用的是4.9-2016.02
```c
Parent Directory
folder	4.9-2016.02	-		
folder	4.9-2017.01	-		
folder	5.1-2015.08	-		
folder	5.2-2015.11	-		
folder	5.2-2015.11-1	-		
folder	5.2-2015.11-2	-		
folder	5.3-2016.02	-		
folder	5.3-2016.05	-		
folder	5.4-2017.01	-		
folder	5.4-2017.05	-		
folder	5.5-2017.10	-		
folder	6.1-2016.08	-		
folder	6.2-2016.11	-		
folder	6.3-2017.02	-		
folder	6.3-2017.05	-		
folder	6.4-2017.08	-		
folder	6.4-2017.11	-		
folder	6.4-2018.05	-		
folder	6.5-2018.12	-		
folder	7.1-2017.05	-		
folder	7.1-2017.08	-		
folder	7.2-2017.11	-		
folder	7.3-2018.05	-		
folder	7.4-2019.02	-		
folder	7.5-2019.12
```

比如我们选择4.9-2016.02这个版本查看
https://releases.linaro.org/components/toolchain/binaries/4.9-2016.02/

arm开头的是arm32使用的，aarch64开头的是arm64使用的，带hf的是支持浮点型的

 - aarch64-linux-gnu
 - arm-linux-gnueabi

	

 - arm-linux-gnueabihf

#### 2、toolchains中的命令介绍
toolchains支持众多命令，列举如下:

```c
gcc-linaro-4.9-2015.02-3-x86_64_aarch64-linux-gnu/bin$ ls
aarch64-linux-gnu-addr2line  aarch64-linux-gnu-cpp        aarch64-linux-gnu-gcc-ar
aarch64-linux-gnu-gfortran  aarch64-linux-gnu-objcopy  aarch64-linux-gnu-strings
aarch64-linux-gnu-ar         aarch64-linux-gnu-elfedit    aarch64-linux-gnu-gcc-nm
aarch64-linux-gnu-gprof     aarch64-linux-gnu-objdump  aarch64-linux-gnu-strip
aarch64-linux-gnu-as         aarch64-linux-gnu-g++        aarch64-linux-gnu-gcc-ranlib
aarch64-linux-gnu-ld        aarch64-linux-gnu-ranlib   gdbserver
aarch64-linux-gnu-c++        aarch64-linux-gnu-gcc        aarch64-linux-gnu-gcov
aarch64-linux-gnu-ld.bfd    aarch64-linux-gnu-readelf  runtest
aarch64-linux-gnu-c++filt    aarch64-linux-gnu-gcc-4.9.3  aarch64-linux-gnu-gdb
aarch64-linux-gnu-nm        aarch64-linux-gnu-size
```

**部分命令介绍：**

 - aarch64-linux-gnu-gcc  这就是交叉编译器，将源文件编译成elf可执行文件
 - aarch64-linux-gnu-strip  删除elf中的符号表，生成干净的可执行文件
 - aarch64-linux-gnu-objdump  将elf文件反汇编，输出dump文件

#### 3、objdump的详细使用
查看help信息

```c
bin$ ./aarch64-linux-gnu-objdump
Usage: ./aarch64-linux-gnu-objdump <option(s)> <file(s)>
 Display information from object <file(s)>.
 At least one of the following switches must be given:
  -a, --archive-headers    Display archive header information
  -f, --file-headers       Display the contents of the overall file header
  -p, --private-headers    Display object format specific file header contents
  -P, --private=OPT,OPT... Display object format specific contents
  -h, --[section-]headers  Display the contents of the section headers
  -x, --all-headers        Display the contents of all headers
  -d, --disassemble        Display assembler contents of executable sections
  -D, --disassemble-all    Display assembler contents of all sections
  -S, --source             Intermix source code with disassembly
  -s, --full-contents      Display the full contents of all sections requested
  -g, --debugging          Display debug information in object file
  -e, --debugging-tags     Display debug information using ctags style
  -G, --stabs              Display (in raw form) any STABS info in the file
  -W[lLiaprmfFsoRt] or
  --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,
          =frames-interp,=str,=loc,=Ranges,=pubtypes,
          =gdb_index,=trace_info,=trace_abbrev,=trace_aranges,
          =addr,=cu_index]
                           Display DWARF info in the file
  -t, --syms               Display the contents of the symbol table(s)
  -T, --dynamic-syms       Display the contents of the dynamic symbol table
  -r, --reloc              Display the relocation entries in the file
  -R, --dynamic-reloc      Display the dynamic relocation entries in the file
  @<file>                  Read options from <file>
  -v, --version            Display this program's version number
  -i, --info               List object formats and architectures supported
  -H, --help               Display this information
```

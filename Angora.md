## 使用文档&常见问题

### Angora编译

使用build_bench_Angora.sh来编译，build_bench_Angora.sh中有注释hh，编译Track时候很慢很慢（编译objdump大概会用20min左右）

### Angora fuzz

#### 使用tool-script跑（推荐）

``` shell
/tool_script/run_Angora.sh objdump-2.31.1-2018-17360 "--dwarf-check -C -g -f -dwarf -x @@" file 60
```

该脚本与DAFL的其他run脚本类似，会先将seed，二进制复制到/box目录下，然后开启Angora Fuzzer

#### 直接跑

```
/ruc-angora/angora_fuzzer -i /benchmark/seed/objdump-2.31.1-2018-17360-best_case/ -o /tmp/objdump-2018-17360-best-case-1 -t /benchmark/bin/Angora/track/objdump-2.31.1-2018-17360 -- /benchmark/bin/Angora/fast/objdump-2.31.1-2018-17360 --dwarf-check -C -g -f -dwarf -x @@
```

- -i: 种子目录

- -o: 目标输出目录，如果目标目录已经存在，Angora会报错，然后退出

- -t: 编译上了污点分析的二进制，track版本

- --: 中间有个奇怪的--，我也不知道这是用来干啥的，但是不加的话，参数解析会出错

- -- 之后跟的是fast版本二进制及其命令行参数

如果出现以下错误：

**WARN**  angora::fuzz_main   > There is **none constraint** in the seeds, please ensure the inputs are vaild in the seed directory, or the program is ran correctly, or the read functions have b

een marked as source.

**INFO**  angora::depot::dump > dump constraints and chart..

说明Track二进制无法生成对应的污点分析程序


### CCDL

```
python3 /ccdl/CCDL.py -q ./queue/ -o ./objdump-2017-8398-best-case-2-ccdl-out -t /benchmark/target/stack-trace/objdump/2017-8398 -b /benchmark/bin/CCDL/objdump-2017-8398 -a "-W %{AFL_FILE}" --asan-binary-path /benchmark/bin/ASAN/objdump-2017-8398 --crash ./crashes/ -l zh
```

- -q: fuzzer输出目录的queue目录
- -o: ccdl结果输出目录
- -t: 例子的调用栈
- -b: 加了-fprofile-instr-generate -fcoverage-mapping编译参数的二进制文件路径，我全部放到了/benchmark/bin/CCDL下，可以通过运行build_bench_ccdl.sh来编译得到
- -a: 例子的命令行参数
- --asan-binary-path: 编译了ASAN的二进制程序
- --crash: fuzzer输出目录的crashes目录
- -l: ccdl语言，默认为英文，这里设置为中文

### 常见问题

#### 问题1

```
checking for BZ2_bzBuffToBuffCompress in -lbz2... no
configure: error: Could not find bz2 library - please install libbz2-dev
```

修改~/.bashrc的LD_LIBRARY_PATH加上/usr/lib:/usr/lib64

#### 问题2

遇到multiple definition

编译选项中添加“--allow-multiple-definition”

#### 问题3

遇到undefined reference

修改rules

例如：

```
/bin/bash ./libtool --tag=CC   --mode=link /ruc-angora/bin/angora-clang -W -Wall -Wstrict-prototypes -Wmissing-prototypes -Wshadow -Werror -I./../zlib -g -fno-omit-frame-pointer -Wno-error -Wl,--allow-multiple-definition  -fno-inline   -o as-new app.o as.o atof-generic.o compress-debug.o cond.o depend.o dwarf2dbg.o dw2gencfi.o ecoff.o ehopt.o expr.o flonum-copy.o flonum-konst.o flonum-mult.o frags.o hash.o input-file.o input-scrub.o listing.o literal.o macro.o messages.o output-file.o read.o remap.o sb.o stabs.o subsegs.o symbols.o write.o tc-i386.o obj-elf.o atof-ieee.o  ../opcodes/libopcodes.la ../bfd/libbfd.la ../libiberty/libiberty.a
libtool: link: /ruc-angora/bin/angora-clang -W -Wall -Wstrict-prototypes -Wmissing-prototypes -Wshadow -Werror -I./../zlib -g -fno-omit-frame-pointer -Wno-error -Wl,--allow-multiple-definition -fno-inline -o as-new app.o as.o atof-generic.o compress-debug.o cond.o depend.o dwarf2dbg.o dw2gencfi.o ecoff.o ehopt.o expr.o flonum-copy.o flonum-konst.o flonum-mult.o frags.o hash.o input-file.o input-scrub.o listing.o literal.o macro.o messages.o output-file.o read.o remap.o sb.o stabs.o subsegs.o symbols.o write.o tc-i386.o obj-elf.o atof-ieee.o  ../opcodes/.libs/libopcodes.a ../bfd/.libs/libbfd.a -L/benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib -lz ../libiberty/libiberty.a
/usr/bin/ld: /benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/libz.a(libz_a-deflate.o): in function `deflateResetKeep':
/benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/deflate.c:416: undefined reference to `__dfsw_crc32'
/usr/bin/ld: /benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/libz.a(libz_a-deflate.o): in function `deflate':
/benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/deflate.c:691: undefined reference to `__dfsw_crc32'
/usr/bin/ld: /benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/deflate.c:727: undefined reference to `__dfsw_crc32'
/usr/bin/ld: /benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/deflate.c:770: undefined reference to `__dfsw_crc32'
/usr/bin/ld: /benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/deflate.c:781: undefined reference to `__dfsw_crc32'
/usr/bin/ld: /benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/libz.a(libz_a-deflate.o):/benchmark/RUNDIR-binutils-2.26/binutils-2.26/zlib/deflate.c:799: more undefined references to `__dfsw_crc32' follow
clang-12: error: linker command failed with exit code 1 (use -v to see invocation)
make[4]: *** [Makefile:770: as-new] Error 1
make[4]: Leaving directory '/benchmark/RUNDIR-binutils-2.26/binutils-2.26/gas'
make[3]: *** [Makefile:2211: all-recursive] Error 1
make[3]: Leaving directory '/benchmark/RUNDIR-binutils-2.26/binutils-2.26/gas'
make[2]: *** [Makefile:698: all] Error 2
make[2]: Leaving directory '/benchmark/RUNDIR-binutils-2.26/binutils-2.26/gas'
make[1]: *** [Makefile:4883: all-gas] Error 2
make[1]: Leaving directory '/benchmark/RUNDIR-binutils-2.26/binutils-2.26'
make: *** [Makefile:846: all] Error 2
cp: cannot stat 'RUNDIR-binutils-2.26/cxxfilt': No such file or directory
```

我们发现，报错信息显示__dfsw_crc32未被定义，参考doc/example.md，我们需要修改一下bin/rules目录下的文件（理论上rules目录下随便找一个文件修改就行，这里我们修改zlib_abilist.txt，将102行的fun:crc32=custom 改为 fun:crc32=discard

![image](https://github.com/user-attachments/assets/921e1e09-95e1-4e63-907f-62a004845890)


### 原文地址

https://flowus.cn/killuayz/share/5729a544-6654-4051-854e-d8f9dd8eb8a9?code=10S3CA

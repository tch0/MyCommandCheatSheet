# Linux命令速查表

## GCC编译链接

- 预处理：`gcc -E`
- 编译到汇编源文件：`gcc -S`
- 编译到目标文件：`gcc -c`
- 生成静态库：`ar rcs libterget.a file1.o file2.o ...`
- 链接静态库：`gcc -static -o xxx filexxx.o -L. -ltarget`
- 链接静态库：`gcc -static -o xxx filexxx.o ./libtarget.a`
- 生成动态库：`gcc -shared -fpic -l libtarget.so file1.c file2.o`
- 链接动态库：`gcc -o prog file1.c file2.o ./libtarget.so`
- 需要使用`dlopen`等接口在程序中加载动态库：
    - 则需要在编译可执行文件时加入`-ldl`选项将`libdl.so`链接进来：`gcc -o prog file1.c file2.o -ldl`
    - 如果要导出可执行文件中的全局符号给`dlopen`动态加载的动态库用那么需要在编译可执行文件时加上选项`-rdynamic`。
    - 有无导出部分符号替代`-rdynamic`的机制？

## 目标文件处理工具

- `ar`：创建静态库，插入、删除、列出、提取成员。
- `strings`：列出目标文件中所有可打印字符串。
- `strip`：从目标文件中删除符号表信息。
- `mm`：列出目标文件符号表中定义的符号。
- `size`：列出目标文件中节的名称和大小。
- `readelf`：显式一个目标文件的完整结构，包括ELF头中编码的所有信息。
    - `-a`：所有。
    - `-h`：ELF头。
    - `-l`：程序头/段头部表。
    - `-S`：节头部表。
    - `-e`：`-h -l -S`，所有头。
    - `-s`：符号表。
    - `-r`：重定位表。
    - 更多见帮助。
- `objdump`：显式一个目标文件中的所有信息，最大作用是反汇编。
    - `-a`：归档文件头信息。
    - `-f`：全面的文件头信息。
    - `-h`：节头部表内容。
    - `-x`：所有头。
    - `-d`：反汇编可执行的节。
    - `-D`：反汇编所有节内容。
    - `-S`：在反汇编时混合插入源码。
    - `-s`：所有节的的全部内容。
    - `-g`：对象文件中的调试信息。
    - `-t`：符号表内容。
    - `-r`：重定位信息。
- `ldd`：列出一个可执行文件在运行时所需要的共享库。

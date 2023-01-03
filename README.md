# Linux命名速查表

## GCC编译链接

- 预处理：`gcc -E`
- 编译到汇编源文件：`gcc -S`
- 编译到目标文件：`gcc -c`
- 生成静态库：`ar rcs libterget.a file1.o file2.o ...`
- 链接静态库：`gcc -static -o xxx filexxx.o -L. -ltarget`
- 链接静态库：`gcc -static -o xxx filexxx.o ./libtarget.a`


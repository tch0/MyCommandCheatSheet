<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Linux命令速查表](#linux%E5%91%BD%E4%BB%A4%E9%80%9F%E6%9F%A5%E8%A1%A8)
  - [GCC编译链接](#gcc%E7%BC%96%E8%AF%91%E9%93%BE%E6%8E%A5)
  - [目标文件处理工具](#%E7%9B%AE%E6%A0%87%E6%96%87%E4%BB%B6%E5%A4%84%E7%90%86%E5%B7%A5%E5%85%B7)
  - [进程操作工具](#%E8%BF%9B%E7%A8%8B%E6%93%8D%E4%BD%9C%E5%B7%A5%E5%85%B7)
  - [网络监测](#%E7%BD%91%E7%BB%9C%E7%9B%91%E6%B5%8B)
  - [RPM包管理器与YUM软件仓库](#rpm%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8%E4%B8%8Eyum%E8%BD%AF%E4%BB%B6%E4%BB%93%E5%BA%93)
  - [Systemd服务管理](#systemd%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86)
  - [dpkg包管理器与APT软件仓库](#dpkg%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8%E4%B8%8Eapt%E8%BD%AF%E4%BB%B6%E4%BB%93%E5%BA%93)
  - [传统服务管理](#%E4%BC%A0%E7%BB%9F%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86)
  - [常用系统工作命令](#%E5%B8%B8%E7%94%A8%E7%B3%BB%E7%BB%9F%E5%B7%A5%E4%BD%9C%E5%91%BD%E4%BB%A4)
  - [系统状态监测](#%E7%B3%BB%E7%BB%9F%E7%8A%B6%E6%80%81%E7%9B%91%E6%B5%8B)
  - [工作目录切换](#%E5%B7%A5%E4%BD%9C%E7%9B%AE%E5%BD%95%E5%88%87%E6%8D%A2)
  - [文本文件编辑](#%E6%96%87%E6%9C%AC%E6%96%87%E4%BB%B6%E7%BC%96%E8%BE%91)
  - [文件目录管理](#%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95%E7%AE%A1%E7%90%86)
  - [打包与压缩](#%E6%89%93%E5%8C%85%E4%B8%8E%E5%8E%8B%E7%BC%A9)
  - [搜索](#%E6%90%9C%E7%B4%A2)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Linux命令速查表

资料：
- [鸟哥的Linux私房菜](https://linux.vbird.org/)
- [Linux就该这么学](https://www.linuxprobe.com/)
- [The Linux Command Line](http://linuxcommand.org/tlcl.php), [中文版](https://www.kancloud.cn/thinkphp/linux-command-line/39431)
- [The Debian Administrator's Handbook](https://debian-handbook.info/browse/stable/index.html)
- [Linux Command Library](https://linuxcommandlibrary.com/)，一个Linux命令速查网站。
- [Arch Wiki](https://wiki.archlinuxcn.org/wiki/%E7%9B%AE%E5%BD%95)

目的：
- 不追求全面和具体，覆盖常用场景能直接抄就行。
- 未包含的东西请借助网络搜索，具体的命令请细节参考man手册。

## GCC编译链接

- 预处理：`gcc -E`
- 编译到汇编源文件：`gcc -S`
- 编译到目标文件：`gcc -c`
- 生成静态库：`ar rcs libterget.a file1.o file2.o ...`
- 链接静态库：`gcc -static -o xxx filexxx.o -L. -ltarget`
- 链接静态库：`gcc -static -o xxx filexxx.o ./libtarget.a`
- 生成动态库：`gcc -shared -fpic -o libtarget.so file1.c file2.o`
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
- `readelf`：显示一个目标文件的完整结构，包括ELF头中编码的所有信息。
    - `-a`：所有。
    - `-h`：ELF头。
    - `-l`：程序头/段头部表。
    - `-S`：节头部表。
    - `-e`：`-h -l -S`，所有头。
    - `-s`：符号表。
    - `-r`：重定位表。
    - 更多见帮助。
- `objdump`：显示一个目标文件中的所有信息，最大作用是反汇编。
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

## 进程操作工具

- `strace`：打印一个正在运行的程序和它的子进程调用的每个系统调用轨迹。
- `ps`：列出系统中的进程，包括僵死进程。
- `top`：打印出关于当前进程资源使用的信息。
- `pmap`：显示进程的内存映射。
- `/proc`：一个虚拟文件系统，以ASCII形式输出大量内核数据结构的内容。用户程序可以读取这些内容，比如`cat /proc/cpuinfo`显示CPU信息。

## 网络监测

- `nslookup`：查询域名对应的IP。
- `cat /etc/services`：互联网知名服务使用的端口与协议。

## RPM包管理器与YUM软件仓库

- 适用于RHEL/CentOS/Fedora等使用RPM包管理器管理软件的发行版。
- RPM包管理器：
    - `rpm -ivh filename.rpm`：安装软件。
    - `rpm -Uvh filename.rpm`：升级软件。
    - `rpm -e filename.rpm`：卸载软件。
    - `rpm -qpi filename.rpm`：查询软件描述信息。
    - `rpm -qpl filename.rpm`：查询软件文件信息。
    - `rpm -qf filename`：查询文件属于哪一个软件包。
        - 其中的`filename`是软件包安装后提供的文件，通过路径给定，这个路径可以通过`which/whereis command`来查询。
- YUM软件仓库：
    - 通常我们都直接从软件仓库中安装软件，而不是下载了`rpm`软件包之后手动安装。
    - `yum makecache`：从yum仓库中下载所有软件包的元信息，缓存在本地。
    - `yum clean all`：清除所有仓库缓存。
    - `yum repolist all`：列出所有仓库。
    - `yum list all`：列出仓库中所有软件包，`yum list`还可以列出其他，通过补全查看。
    - `yum info package-name`：查看软件包信息。
    - `yum install package-name`：安装软件包。
    - `yum reinstall package-name`：重新安装软件包。
    - `yum update package-name`：升级软件包。
    - `yum remove package-name`：移除软件包。
    - `yum check-update`：检查可更新的软件包。
    - `yum grouplist`：查看系统中已安装的软件包组。
    - `yum groupintall package-group`：安装指定软件包组。
    - `yum groupremove package-group`：移除软件包组。
    - `yum groupinfo package-group`：查看指定软件包组信息。

## Systemd服务管理

- 适用于RHEL/CentOS/Fedora/Ubuntu/Debian等默认使用Systemd管理服务的发行版。
- `systemctl start foo.service`：启动
- `systemctl restart foo.service`：重启
- `systemctl stop foo.service`：停止
- `systemctl reload foo.service`：重新加载配置文件（不终止服务）
- `systemctl status foo.service`：查看服务状态
- `systemctl enable foo.service`：开机自启
- `systemctl disable foo.service`：开机不自启
- `systemctl is-enabled foo.service`：查看特定服务是否开机自启
- `systemctl list-unit-files --type=service`：查看各个级别服务的启动与禁止情况
- [Systemd 入门教程：命令篇](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

## dpkg包管理器与APT软件仓库

- 适用于Debian系列的发行版，比如Debian/Ubuntu等。
- dpkg包管理器：
    - 是Debian Packager的简写，为Debian系统的专用包管理器。
    - Debian系统的包一般以`.deb`作为后缀。
    - 如果需要手动安装`.deb`包而不是直接从APT软件仓库下载，就需要使用`dpkg`命令。
- `dpkg`命令：
    - 安装前需要给出包的完整路径：
    - `dpkg -i package-name.deb`：安装软件包。
    - `dpkg -c package-name.deb`：列出包的内容。
    - `dpkg --unpack package-name.deb`：解开包的内容。
    - 安装后则只需要给出包名：
    - `dpkg -l package-name`：显示包的版本。
    - `dpkg -r package-name`：移除包。
    - `dpkg -P package-name`：移除包及其配置文件。
    - `dpkg -s package-name`：查看包的安装状态信息。
    - `dpkg -S package-name`：列出包安装后所属的所有文件。
    - `dpkg --configure package-name`：配置软件包。
- APT软件仓库：
    - APT is Advanced Packaging Tool.
    - 在apt仓库中一个令人迷惑的点就是有`apt/apt-get`两条命令。
    - 他们的大部分语法基本相同，区别是`apt`设计上就是为了用于命令行交互用的。
    - 现在来说`apt-get`仅应该用在脚本中。
    - `apt`的功能大概相当于`apt-get/apt-cache/apt-config`的集合。
    - 注意的是`apt-get`并未弃用，作用普通用户，应当使用`apt`。
- `apt`命令：
    - `apt install`：安装软件包。
    - `apt remove`：移除软件包。
    - `apt purge`：移除软件包及其配置文件。
    - `apt update`：更新可用软件包列表。
    - `apt upgrade`：通过 安装/升级 更新系统。
    - `apt autoremove`：卸载所有自动安装且不再使用的软件包。
    - `apt full-upgrade`：通过 卸载/安装/升级 来更新系统。
    - `apt search`：搜索软件包。
    - `apt show`：显示软件包细节。
    - `apt list`：根据名称列出软件包。
    - `apt edit-sources`：编辑源。


## 传统服务管理

- 指System V/upstart等服务管理，使用`chkconfig/service`等命令以及脚本管理服务。
- 在默认使用Systemd的发行版上，为了兼容性这些命令可能依然可用，但是可能是systemctl命令的对应别名。
- 至于传统服务管理和Systemd服务管理孰好孰坏，争论很多，服务管理还有更多细节，这里不过多赘述。
    - 总体来说systemd大而全，被抨击不符合UNIX所有东西小而精做好自己的事情的哲学。
    - 传统的服务管理则使用脚本来管理服务，更符合UNIX哲学。
    - 现在的发行版大多都是用systemd替换了传统服务管理。
- System V服务管理，使用`chkconfig`命令：
    - `chkconfig --list`：列出所有服务。
    - `chkconfig --add service_name`：添加服务。
    - `chkconfig --del service_name`：删除服务。
    - `chkconfig service_name on`：开机自启。
- upstart管理服务，用在早期ubuntu中：
    - `service script start`：启动服务。
    - `service script stop`：停止服务。
    - `service script restart`：重启服务。
    - `service script status`：查看服务状态。
    - `service --status-all`：显示所有系统服务列表。`+ -`分别表示正在运行和关闭。
- 这里仅做参考，不一定准确，现在可能已经用不到了。

## 常用系统工作命令

- `echo [string | $var]`：输出字符串或者变量的值，比如`echo $SHELL`输出当前的shell。
- `date`：显示及设置系统时间。
- `reboot`：重启计算机。
- `poweroff`：关闭计算机。
- `wget`：在终端中下载网络文件。
    - 下载一个简单文件：`wget url`。
    - 递归下载一个网站内所有数据及文件：`wget -r -p url`。
- `ps`：查看系统中进程状态。
    - `-a`显示所有。
    - `-u`显示用户及其他详细信息。
    - `-x`显示没有控制终端的进程。
    - 最常用形式：`ps -aux`。
- `top`：动态监视进程活动和系统负载信息。类似于Windows中的任务管理器。
- `pidof`：查看某个进程的PID（进程ID，唯一标识进程的一个ID）。
- `kill [options] [pid]`：终止某个进程，传入的PID可以通过`pidof ps`命令查询。
- `killall [options] [service name]`：终止某个服务对应的全部进程。复杂软件的服务程序会有多个进程协同为用户提供服务，通过`pidof`会输出所有PID，通过`killall`就不需要再去查询PID并一个一个输入了。
- TIP：终端执行一个程序时，还在执行中的时候按Ctrl+C可以终止这个程序，输入命令后跟`&`会让其进入后台执行。

## 系统状态监测

- `ifconfig [options] [network-device]`：查看本机的网卡配置与网络信息。
    - 现在一定程度上已经被`ip addr`命令替代，在某些发行版上不再内置，如果无法识别该命令，需要先安装`net-tools`软件包。
    - 可以查看网卡名称、IP地址、子网掩码、MAX地址、最多传输单元、接受(RX)和发送(TX)的数据包个数、累计流量等信息。
- `uname [-a]`：查看系统内核与系统版本等信息。
    - 主要包括内核名称、主机名、内核发行版本、节点名、系统时间、硬件名称、硬件平台、处理器类型、操作系统名称等信息。
    - 如果需要查看当前系统详细信息，则不同发行版不同命令：
    - CentOS/RHEL/etc：`cat /etc/redhat-release` `cat /etc/centos-release`。
    - Ubuntu：`lsb_release -a`。
- `uptime`：查看系统负载信息。
    - 显示当前系统时间、系统已运行时间、启用终端数量、1/5/15分钟内的平均负载。负载越低越好，尽量不要长期超过1。这些信息在top命令第一行中都有。
- `free [-h]`：查看系统中内存使用信息，`top`命令中同样能看到这些信息。
- `who`：查看当前登入主机的用户终端信息。
    - 包括用户名、登入终端、登入时间等信息。
    - 可以通过`whoami`命令查看当前用户，仅输出用户名。
- `last [options]`：查看系统的所有登录记录。
- `history`：显示历史执行过的命令。
    - 每个命令都会有一个编号，可以通过`!num`来执行对应编号的命令。
    - 最多1000条，通过`/etc/profile`中的`HISTSIZE`变量指定，可以修改。
    - `-c`参数清除历史。
    - 历史记录保存在`~/.bash_history`文件中。
- `sosreport`：收集系统配置即架构信息并输出诊断文档。
    - 如果不可用，需要先安装。
    - 当系统出现故障需要联系技术人员时，一般都先执行这个命令来收集系统运行状态和服务配置信息，以便技术支持人员能够提前了解信息。
    - 生成后保存在`/tmp /var/tmp`等临时目录，会计算校验和（md5/sha256）并保存在同目录文件中。
    - 需要`sudo`执行。

## 工作目录切换

## 文本文件编辑

## 文件目录管理

## 打包与压缩

## 搜索
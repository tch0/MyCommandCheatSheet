<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [各种Linux发行版基础配置](#%E5%90%84%E7%A7%8Dlinux%E5%8F%91%E8%A1%8C%E7%89%88%E5%9F%BA%E7%A1%80%E9%85%8D%E7%BD%AE)
  - [Ubuntu](#ubuntu)
    - [换源](#%E6%8D%A2%E6%BA%90)
    - [代理](#%E4%BB%A3%E7%90%86)
    - [软件安装](#%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85)
  - [CentOS](#centos)
    - [安装](#%E5%AE%89%E8%A3%85)
    - [初始配置](#%E5%88%9D%E5%A7%8B%E9%85%8D%E7%BD%AE)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 各种Linux发行版基础配置

## Ubuntu

### 换源

- 不要直接搜索网络上的教程抄，这样配出来的软件源通常不是最新的。
- 直接去镜像站找对应教程比如ustc：https://mirrors.ustc.edu.cn/help/ubuntu.html
- 如果下载软件发现版本低，比如gcc只有8.1，去检查一下源。

### 代理

- 按需配置
```
export all_proxy=socks://ip:port
export http_proxy=http://ip:port
export https_proxy=https://ip:port
```

### 软件安装

- 安装openssh-server以支持远程连接：
```
sudo apt install openssh-server
```

## CentOS

版本：CentOS 7

### 安装

- 安装在VMware虚拟机中时，选择经典安装，以自己选择系统软件。比如选Server With GUI。

### 初始配置

- 更换软件源：https://mirrors.ustc.edu.cn/help/centos.html
- 代理（或者在设置中配置也可以）：
```
export all_proxy=socks://ip:port
export http_proxy=http://ip:port
export https_proxy=https://ip:port
```
- 将服务器的软件包信息缓存在本地：
```
yum makecache
```
- 更新yum：
```
sudo yum update -y
```

## WSL

WSL中查看宿主机IP：
```
cat /etc/resolv.conf
```
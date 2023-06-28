<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Git命令速查](#git%E5%91%BD%E4%BB%A4%E9%80%9F%E6%9F%A5)
  - [子模块](#%E5%AD%90%E6%A8%A1%E5%9D%97)
  - [配置代理](#%E9%85%8D%E7%BD%AE%E4%BB%A3%E7%90%86)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Git命令速查

- [git命令速查](https://training.github.com/downloads/zh_CN/github-git-cheat-sheet/)
- [git可视化命令速查](https://ndpsoftware.com/git-cheatsheet.html#loc=stash;)

## 子模块

- [文档](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
- 在现存库中添加一个子模块：`git submodule add -b the-branch git@host:username/project.git path/to/submodule`
- 签出子模块到特定提交或者标签：`git checkout tag/commit`，在主仓库中提交之后即可将子模块固定到特定提交。
- 修改子模块的远程路径：`git submodule set-url sub-module-name new-url`
- 查看子模块状态：`git submodule status`
- `.gitmodules`需要跟踪，不要自行修改其内容。
- 默认签出主仓库后子模块的目录中是空的，需要更新子模块。克隆仓库时添加`--recurse-submodules`选项则会同时克隆子模块。
- 初始化与更新子模块：`git submodule init`和`git submodule update`，合起来`git submodule update --init`
- 删除子模块：
  - 清除缓存：`git rm --cached submodule-name`。
  - 移除子模块源码目录 `rm -rf path/to/submodule`。
  - 修改`.gitmodules`对应条目。
  - 删除`.git/config`中对应条目。
  - 删除`.git`目录中的对应子模块目录 `rm .git/modules/submodule-name`。
- 对子模块进行修改提交会自动更新主仓库中的子模块版本。

子模块的典型场景：
- 用到了别人的第三方库，将其整合到自己的项目中，方便追踪最新的更新或者固定到特定版本，将第三方库作为子模块签出到项目中。
- 为了防止远程仓库删除，可以先fork之后再添加子模块。
- 因为github国内访问很慢，可以先将仓库导入到gitee之后，从gitee添加子模块，添加完成后再将地址修改为github。

## 配置代理

- 参考：https://zhuanlan.zhihu.com/p/481574024

- https代理：
```shell
#使用http代理 
git config --global http.proxy http://127.0.0.1:58591
git config --global https.proxy https://127.0.0.1:58591
#使用socks5代理
git config --global http.proxy socks5://127.0.0.1:51837
git config --global https.proxy socks5://127.0.0.1:51837
```
- 只对`github.com`进行代理，避免对其他网站比如`gitee.com`进行代理：
```shell
#使用socks5代理（推荐）
git config --global http.https://github.com.proxy socks5://127.0.0.1:51837
#使用http代理（不推荐）
git config --global http.https://github.com.proxy http://127.0.0.1:58591
```
- 直接修改文件`~/.gitconfig`：
```
[http "https://github.com"]
	proxy = socks5://127.0.0.1:33211
[https "https://github.com"]
	proxy = socks5://127.0.0.1:33211
```
- 取消代理，删除文件对应内容或者
```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```
- ssh代理：修改`~/.ssh/config`，添加内容：
```
#Windows用户，注意替换你的端口号和connect.exe的路径
ProxyCommand "C:\APP\Git\mingw64\bin\connect" -S 127.0.0.1:51837 -a none %h %p

#MacOS用户用下方这条命令，注意替换你的端口号
#ProxyCommand nc -v -x 127.0.0.1:51837 %h %p

Host github.com
  User git
  Port 22
  Hostname github.com
  # 注意修改路径为你的路径
  IdentityFile "C:\Users\Your_User_Name\.ssh\id_rsa"
  TCPKeepAlive yes

Host ssh.github.com
  User git
  Port 443
  Hostname ssh.github.com
  # 注意修改路径为你的路径
  IdentityFile "C:\Users\Your_User_Name\.ssh\id_rsa"
  TCPKeepAlive yes
```
- 其中https和socks5代理端口需要去代理软件中查看。

- **为WSL2配置使用本机代理**：执行`cat /etc/resolv.conf`查看本机IP，替换上面步骤中的IP即可。

- **为虚拟机中的Ubuntu配置使用本机代理**：
  - 本机执行`ipconfig`找到本机的局域网IP，虚拟机网络连接模式选择桥接。
  - 按照上面的步骤配置即可，IP替换为上面的本机IP。
  - 虚拟机中更好的方式是配置为全局的，这样浏览器以及其他命令也能用，也就不需要单独为git配置了：设置-网络-网路代理-手动然后配置socks主机即可。效果其实就是将`all_proxy=socks://ip:port/`添加到了环境中（IP和端口是上面的）。
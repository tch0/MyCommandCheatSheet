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
- 对子模块进行修改提交会自动更新主仓库中的子模块版本。

子模块的典型场景：
- 用到了别人的第三方库，将其整合到自己的项目中，方便追踪最新的更新或者固定到特定版本，将第三方库作为子模块签出到项目中。
- 为了防止远程仓库删除，可以先fork之后再添加子模块。
- 因为github国内访问很慢，可以先将仓库导入到gitee之后，从gitee添加子模块，添加完成后再将地址修改为github。

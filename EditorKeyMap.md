# 跨编辑器快捷键映射指南


## 编辑器快捷键

- 下面表中都是编辑器中最重要的几个功能。
- 第一次使用一个编辑器时首先找到这些功能，将这些快捷键映射加进去（最好是添加而非替换除非有冲突），即可跨编辑器使用同一套快捷键，而不需要记忆多套。
- 一些编辑器可能不提供完全相同的功能，功能有强弱，但近似的功能应该都有。
- 这里再提供VS和VS Code的添加指南。

|功能|快捷键|来源|VS Code添加到|VS添加到
|:-|:-|:-|:-|:-
|关闭当前文件|**Ctrl+W**|浏览器|查看:关闭编辑器|窗口.关闭文档窗口
|声明和定义之间互相跳转</p>以及转到头文件|**Alt+G**|VAX|转到定义|编辑.转到
|转到当前文件中的某个符号|**Alt+M**|VAX|转到编辑器中的符号|编辑.转至“成员”...（需要固定到当前文件范围，并且不能列出当前文件中的符号列表，不是很好用，但也找不到其他更合适的了）
|查找某个符号的所有引用|**Alt+Shift+F**|VAX|引用:查找所有引用|编辑.查找所有引用
|转到文件|**Alt+Shift+O**|VAX|转到文件|编辑.转到文件
|转到符号|**Alt+Shift+S**|VAX|转到工作区中的符号|编辑.转到符号
|重命名符号|**Alt+Shift+R**|VAX|重命名符号|编辑.重命名
|将下一个匹配项添加到选择|**Ctrl+D**|VS Code|将下一个查找匹配项添加到选择|编辑.插入下一个匹配的文字光标
|声明定义文件之间跳转|**Alt+O**|VS Code|C/C++:切换标头/源|编辑器上下文菜单.代码窗口.切换标题代码文件
|注释行|**Ctrl+K Ctrl+C**|VS/VS Code|添加行注释|编辑.注释选定内容
|取消注释行|**Ctrl+K Ctrl+U**|VS/VS Code|删除行注释|编辑.取消注释选定内容

## 调试快捷键

VS和VS Code的调试快捷键是基本完全相同的，VS Code的某些插件可能有不同的快捷键（比如CMake插件），最好将这些插件、其他IDE也调整为这套快捷键。

|功能|快捷键
|:-|:-
|加断点|**F9**
|启动调试/跳转到下一个断点/继续|**F5**
|启动但不调试（也就是运行）|**Ctrl+F5**
|停止调试|**Shift+F5**
|单步跳过|**F10**
|单步执行|**F11**
|单步跳出|**Shift+F11**
|运行到光标处|**Ctrl+F10**
|删除所有断点|**Ctrl+Shift+F9**
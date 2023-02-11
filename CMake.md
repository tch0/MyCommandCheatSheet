# CMake命令速查


## 配置与构建

- 通用构建流程：
```sh
mkdir build
cd build
cmake .. [-G "generator"] # configure
cmake --build . # build
```
- 选择特定工具构建：
```sh
cmake .. -G "UNIX Makefiles" # Unix
cmake .. -G "MinGW Makefiles" # Windows MinGW
cmake .. -G "Visual Studio 17 2022" # Windows VS2022
...
```
- 帮助：
```sh
cmake --help
```
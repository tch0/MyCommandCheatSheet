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

## 构建类型选择

- 构建类型选择：
```sh
cmake .. [-G "generator"] -DCMAKE_BUILD_TYPE=xxx
```
- 预定义的可选的构建类型：
    - `Debug`：无优化，提供完整调试信息。
    - `Release`：进行优化，无调试信息。
    - `MinSizeRel`：优化，提供最小的二进制大小，无调试信息。
    - `RelWithDebInfo`：进行优化，包含调试信息。
    - 典型的编译参数，gcc为例：
    ```
    //Flags used by the CXX compiler during all build types.
    CMAKE_CXX_FLAGS:STRING=

    //Flags used by the CXX compiler during DEBUG builds.
    CMAKE_CXX_FLAGS_DEBUG:STRING=-g

    //Flags used by the CXX compiler during MINSIZEREL builds.
    CMAKE_CXX_FLAGS_MINSIZEREL:STRING=-Os -DNDEBUG

    //Flags used by the CXX compiler during RELEASE builds.
    CMAKE_CXX_FLAGS_RELEASE:STRING=-O3 -DNDEBUG

    //Flags used by the CXX compiler during RELWITHDEBINFO builds.
    CMAKE_CXX_FLAGS_RELWITHDEBINFO:STRING=-O2 -g -DNDEBUG
    ```
    - 只有`Debug`才有禁用`NDEBUG`宏，调试断言才会生效。
    - 通常调试用`Debug`，发布用`Release`，其他两个看情况可能用到，但一般来说不需要。
    - 默认类型是空的，不是任何特定的构建类型。
- 对于gcc/clang等工具链，生成单一配置的`Makefile`，要构建不同构建类型的版本，最好的做法是新建多个目录，在其中分别生成：
```sh
cmake .. [-G "generator"] -DCMAKE_BUILD_TYPE=xxx
cmake --build .
```
- 对于MSVC，生成多种配置的Visual Studio项目配置文件，不同构建类型直接对应于不同配置，不需要将多个构建类型的项目文件生成到多个目录中。直接在构建时选择配置即可互不干扰的在不同目录中生成不同配置的二进制、调试信息等。（注意这对单一配置的`Makefile`来说没有用。）
```sh
cmake .. [-G "generator"]
cmake --build . --config Debug
```
- 对于其他IDE，大多同MSVC差不多，可以生成多种配置的项目文件，按情况选择即可。
- 进入GUI窗口中编辑cmake缓存：
```sh
cmake-gui -S path/to/source -B path/to/build
```
- 对于单一配置情况的可以直接调用（其中会调用上面的命令）：
```sh
make edit_cache
```

## 安装

- 安装
```sh
cd ./build
cmake ..
cmake --build .
cmake --install .
```
- 不给`--prefix`参数时会安装到默认目录，Windows中会安装到`C:/Program Files (x86)/${PROJECT}`。Unix中则是对应的目录，比如`usr/local/include /usr/local/bin`之类。
- 安装到指定的目录：
```sh
cmake --install . --prefix "install-dir"
```
- 或者在配置cmake时通过指定`CMAKE_INSTALL_PREFIX`（也可以在`CMakeLists.txt`中写死）：
```sh
cmake .. -DCMAKE_INSTALL_PREFIX=your/install/path
```
- 对于多配置的工具链（比如Visual Studio），可以使用`--config`选择安装特定配置：
```sh
cmake --install . --config Release
```
- 单配置的工具链则是在cmake配置时指定构建类型，从而安装该构建类型。
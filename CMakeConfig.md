<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CMake配置速查](#cmake%E9%85%8D%E7%BD%AE%E9%80%9F%E6%9F%A5)
  - [静态/动态库配置](#%E9%9D%99%E6%80%81%E5%8A%A8%E6%80%81%E5%BA%93%E9%85%8D%E7%BD%AE)
  - [编译选项定制](#%E7%BC%96%E8%AF%91%E9%80%89%E9%A1%B9%E5%AE%9A%E5%88%B6)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CMake配置速查

## 静态/动态库配置

典型手法：
- 现在有一个库`Hello`，我们希望它可以编译为动态库或者静态库。
- 在定义库时，不显式声明静态或者共享/动态库，使用选项`BUILD_SHARED_LIBS`来决定。
```CMake
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
add_library(Hello ${HELLO_SOURCES})
if (${BUILD_SHARED_LIBS})
    target_compile_definitions(Hello PRIVATE HELLO_BUILD_SHARED)
endif()
```
- 源文件的API修饰依赖于两个宏`HELLO_SHARED`和`HELLO_BUILD_SHARED`：
  - 构建库时通过CMake配置在编译器选项中定义宏`HELLO_BUILD_SHARED`（或者通过`configure_file`定义在一个配置头文件中）。
  - 使用动态库版本，则需要在包含头文件前定义一个宏`HELLO_SHARED`。
```C++
// in hello.h
#ifdef (_WIN32)
    #if defined(HELLO_SHARED)
        #define HELLOAPI __declspec(dllimport)
    #elif defined(HELLO_BUILD_SHARED)
        #define HELLOAPI __declspec(dllexport)
    #else // static
        #define HELLOAPI
    #endif
#else // non windows
    #define HELLOAPI
#endif
```
- 如果构建并使用静态库，则构建时配置`BUILD_SHARED_LIBS`为`OFF`，并且包含头文件时也不需要定义宏`HELLO_SHARED`。
- `HELLO_SHARED`和`HELLO_BUILD_SHARED`不应该同时被定义，可以做一个判断来处理：
```C++
#if defined(HELLO_SHARED) && defined(HELLO_BUILD_SHARED)
    #error "You must not have both HELLO_SHARED and HELLO_BUILD_SHARED defined"
#endif
```
- 如果这个库不是一个单独的项目，而是一个大项目中的一个子项目，如果编译为动态库，那么每次包含头文件前都需要定义`HELLO_SHARED`未免有点不太合理，这时候按道理来说应该不会将一个库可选地既可以构建动态库版本，也可以构建静态库版本，而是要么使用静态库，要么使用动态库：
  - 如果使用静态库，那么去掉这些东西，没有任何API修饰即可。
  ```C++
  add_library(Hello STATIC ${HELLO_SOURCES})
  ```
  - 如果使用动态库，那么就保证了应该不会使用静态库，源文件中定义改为：
  ```C++
  // in hello.h
  #ifdef (_WIN32)
      #if defined(HELLO_BUILD_SHARED)
          #define HELLOAPI __declspec(dllexport)
      #else
          #define HELLOAPI __declspec(dllimport)
      #endif
  #else // non windows
      #define HELLOAPI
  #endif
  ```
  - CMake配置中改为：
  ```CMake
  add_library(Hello SHARED ${HELLO_SOURCES})
  target_compile_definitions(Hello PRIVATE HELLO_BUILD_SHARED)
  ```

## 编译选项定制

可以将编译选项定制为interface library，通过`target_link_library`以将编译选项传播到其他库：
- 典型警告选项配置：
```CMake
# general compiler flags, C++ standard, warnings, etc.
add_library(general_cxx_compiler_flags INTERFACE)
# C++ standard
target_compile_features(general_cxx_compiler_flags INTERFACE cxx_std_11)
# for different toolchain
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
# warning options for different toolchain
target_compile_options(general_cxx_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-pedantic-errors;-Wformat=2;-Wno-unused-parameter>>"
    "$<${msvc_cxx}:$<BUILD_INTERFACE:/W3>>"
)
```
- 链接选项以及预定义宏等，同理。
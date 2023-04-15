<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CMake配置速查](#cmake%E9%85%8D%E7%BD%AE%E9%80%9F%E6%9F%A5)
  - [静态/动态库配置](#%E9%9D%99%E6%80%81%E5%8A%A8%E6%80%81%E5%BA%93%E9%85%8D%E7%BD%AE)
  - [编译选项定制](#%E7%BC%96%E8%AF%91%E9%80%89%E9%A1%B9%E5%AE%9A%E5%88%B6)
  - [文件拷贝](#%E6%96%87%E4%BB%B6%E6%8B%B7%E8%B4%9D)
  - [检查编译器是否支持某特性](#%E6%A3%80%E6%9F%A5%E7%BC%96%E8%AF%91%E5%99%A8%E6%98%AF%E5%90%A6%E6%94%AF%E6%8C%81%E6%9F%90%E7%89%B9%E6%80%A7)
  - [通过编译代码片段检查是否支持某特性](#%E9%80%9A%E8%BF%87%E7%BC%96%E8%AF%91%E4%BB%A3%E7%A0%81%E7%89%87%E6%AE%B5%E6%A3%80%E6%9F%A5%E6%98%AF%E5%90%A6%E6%94%AF%E6%8C%81%E6%9F%90%E7%89%B9%E6%80%A7)
  - [跨编译器配置](#%E8%B7%A8%E7%BC%96%E8%AF%91%E5%99%A8%E9%85%8D%E7%BD%AE)
  - [为IDE（如VS）配置筛选器](#%E4%B8%BAide%E5%A6%82vs%E9%85%8D%E7%BD%AE%E7%AD%9B%E9%80%89%E5%99%A8)

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

## 文件拷贝

有三种实现方式：
- 第一种：**配置时复制**，使用`cofigure_file`，不止能够复制文件，还能够根据CMake的定义对文件进行配置，一般用于头文件。
```CMake
configure_file(<input> <output>)
```
- 第二种：使用`file(COPY <input-file> DESTINATION <output-dir>)`，同样是**配置时复制**，可以用于头文件、配置文件、甚至库文件等。
```CMake
foreach(file_i ${all_files})
    file(COPY file_i DESTINATION <output-dir>)
endforeach()
```
- 第三种：使用`add_custom_command`，执行命令，可以配置在**构建前、链接前、构建后复制**，可以复制文件或者目录，可以用于配置文件复制。其中可以使用生成器表达式。（下面的例子将`sources`中所有文件在构建`target`成功后复制到`target`的生成目录。）也可以使用`copy_directory`复制目录。
```CMake
add_custom_command(TARGET ${target} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy -t $<TARGET_FILE_DIR:${target}> ${sources}
    COMMAND_EXPAND_LISTS
)
```
- 最后的例子可以通过定义函数方便的简化：
```CMake
function(copy_sources_to_target_file_dir target sources)
    set(sources ${ARGV})
    list(POP_FRONT sources)
    add_custom_command(TARGET ${target} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy -t $<TARGET_FILE_DIR:${target}> ${sources}
        COMMAND_EXPAND_LISTS
    )
endfunction()
```
- 这里不能使用`${CMAKE_CURRENT_BINARY_DIR}`，因为在不同的构建工具中这个目录可能不一样，比如VS的生成目录就有一层`Debug/Release`。为了屏蔽不同构建工具的差异，要么编写逻辑处理，要么使用生成器表达式`$<TARGET_FILE_DIR:xxx>`。

## 检查编译器是否支持某特性

- 可以通过检查`CMAKE_CXX_COMPILE_FEATURES`变量来检查是否支持某个特性。
- 具体的特性包括但不限于：见[这里](https://cmake.org/cmake/help/latest/prop_gbl/CMAKE_CXX_KNOWN_FEATURES.html#prop_gbl:CMAKE_CXX_KNOWN_FEATURES)。
```
cxx_std_98
cxx_template_template_parameters
cxx_std_11
cxx_alias_templates
cxx_alignas
cxx_alignof
cxx_attributes
cxx_auto_type
cxx_constexpr
cxx_decltype
cxx_decltype_incomplete_return_types
cxx_default_function_template_args
cxx_defaulted_functions
cxx_defaulted_move_initializers
cxx_delegating_constructors
cxx_deleted_functions
cxx_enum_forward_declarations
cxx_explicit_conversions
cxx_extended_friend_declarations
cxx_extern_templates
cxx_final
cxx_func_identifier
cxx_generalized_initializers
cxx_inheriting_constructors
cxx_inline_namespaces
cxx_lambdas
cxx_local_type_template_args
cxx_long_long_type
cxx_noexcept
cxx_nonstatic_member_init
cxx_nullptr
cxx_override
cxx_range_for
cxx_raw_string_literals
cxx_reference_qualified_functions
cxx_right_angle_brackets
cxx_rvalue_references
cxx_sizeof_member
cxx_static_assert
cxx_strong_enums
cxx_thread_local
cxx_trailing_return_types
cxx_unicode_literals
cxx_uniform_initialization
cxx_unrestricted_unions
cxx_user_literals
cxx_variadic_macros
cxx_variadic_templates
cxx_std_14
cxx_aggregate_default_initializers
cxx_attribute_deprecated
cxx_binary_literals
cxx_contextual_conversions
cxx_decltype_auto
cxx_digit_separators
cxx_generic_lambdas
cxx_lambda_init_captures
cxx_relaxed_constexpr
cxx_return_type_deduction
cxx_variable_templates
cxx_std_17
cxx_std_20
cxx_std_23
```
- 可以检查某一项是否在该列表中来检查是否支持该特性：
```CMake
list(FIND CMAKE_CXX_COMPILE_FEATURES cxx_std_20 cxx20_supported)
if (cxx20_supported EQUAL -1)
    message("C++ 20 is not supported!")
endif()
```

## 通过编译代码片段检查是否支持某特性

- 某些特性并不在上面的`CMAKE_CXX_COMPILE_FEATURES`的列表中，比如库特性。
- 但是我们可以通过编译一个代码片段来检查某个特性是否得到支持。
- 比如检查`<format>`在当前工具链中是否支持，如果支持就使用标准库版本，如果不支持，那么就将一个包含了手动添加的`format`头文件的目录添加到包含目录中。
```C++
// 3rdparty-install/formatbridge/format
#pragma once
#define FMT_HEADER_ONLY
#include <fmt/format.h>
namespace std
{
    using fmt::format;
    using fmt::format_error;
    using fmt::formatter;
}
```
- 在`CMakeLists.txt`中则通过`check_cxx_source_compiles`函数来检测：
```CMake
include(CheckCXXSourceCompiles)

add_library(format_bridge INTERFACE)
check_cxx_source_compiles("#include <format>\nint main() { return 0; }" format_supported)
if (NOT format_supported)
    target_include_directories(format_bridge INTERFACE ${CMAKE_SOURCE_DIR}/3rdparty-install/include/formatbridge)
    message("## <format> is not support on your compiler yet, use fmt library instead!")
endif()
```
- 然后将`format_bridge`库通过`target_link_libraries`链接到要使用的目标中即可。
- 如果支持了`<format>`，比如MSVC 2022，那么就不会使用第三方的fmt库。如果不支持比如GCC 12.2.0，那么就会转而使用第三方库，非常方便，不需要对源码有任何更改。

## 跨编译器配置

- 配置编译选项基本上前面 编译选项定制 里面的生成器表达式就够用了。
- 但是某些时候还需要能够保存到一个变量中，某些编译器（典型的是`clang-cl`）无法通过上面的生成器表达式得到，还可以通过变量`CMAKE_CXX_COMPILER_ID`来获取。更多细节见[这里](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_ID.html#variable:CMAKE_%3CLANG%3E_COMPILER_ID)。

```CMake
# different compilers options
set(CXX_COMPILER_IS_GCC OFF)            # gcc
set(CXX_COMPILER_IS_CLANG OFF)          # clang
set(CXX_COMPILER_IS_MSVC OFF)           # msvc
set(CXX_COMPILER_IS_CLANG_CL OFF)       # clang-cl
set(CXX_COMPILER_IS_GNU_LIKE OFF)       # gcc, clang, clang-cl
set(CXX_COMPILER_IS_GCC_CLANG OFF)      # gcc, clang
set(CXX_COMPILER_IS_CLANG_ALL OFF)      # clang, clang-cl
set(CXX_COMPILER_IS_MSVC_LIKE OFF)      # msvc, clang-cl
if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    set(CXX_COMPILER_IS_GCC ON)
    set(CXX_COMPILER_IS_GNU_LIKE ON)
    set(CXX_COMPILER_IS_GCC_CLANG ON)
    message("## Compiler: gcc")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    if (CMAKE_CXX_SIMULATE_ID MATCHES "MSVC" AND CMAKE_CL_64)
        set(CXX_COMPILER_IS_CLANG_CL ON)
        set(CXX_COMPILER_IS_GNU_LIKE ON)
        set(CXX_COMPILER_IS_CLANG_ALL ON)
        set(CXX_COMPILER_IS_MSVC_LIKE ON)
        message("## Compiler: clang-cl")
    else ()
        set(CXX_COMPILER_IS_CLANG ON)
        set(CXX_COMPILER_IS_GNU_LIKE ON)
        set(CXX_COMPILER_IS_GCC_CLANG ON)
        set(CXX_COMPILER_IS_CLANG_ALL ON)
        message("## Compiler: clang")
    endif ()
elseif (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    set(CXX_COMPILER_IS_MSVC ON)
    set(CXX_COMPILER_IS_MSVC_LIKE ON)
    message("## Compiler: msvc")
else ()
    message(FATAL_ERROR "Unsupported Compiler!")
endif ()


# general C++ compiler options
add_library(general_cxx_compiler_options INTERFACE)
target_compile_options(general_cxx_compiler_options INTERFACE
    # gcc/clang/clang-cl common options
    $<$<BOOL:${CXX_COMPILER_IS_GNU_LIKE}>:$<BUILD_INTERFACE:
        -Wall
        -Wextra
        -Wshadow
        -Wformat=2
        -Wno-unused-parameter
        -Wno-unused-variable
    >>
    # gcc/clang common options
    $<$<BOOL:${CXX_COMPILER_IS_GCC_CLANG}>:$<BUILD_INTERFACE:
        -pedantic-errors
    >>
    # MSVC/clang-cl common options
    $<$<BOOL:${CXX_COMPILER_IS_MSVC_LIKE}>:$<BUILD_INTERFACE:
        /W3
    >>
)
```
- 编译器原则上来所和操作系统是正交的，某些配置还会取决于操作系统。通常我们只关心`Windows/Unix/Apple`三个操作系统：
```CMake
if (APPLE)
    message(STATUS "MacOs")
elseif (UNIX) # Linux
    message(STATUS "Linux")
elseif (WIN32)
    message(STATUS "Windows")
endif()
```
- 其他的环境相关变量还有：`MINGW CYGWIN MSVC LINUX MSYS BSE`等，见[描述环境的CMake变量](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html#variables-that-describe-the-system)。


## 为IDE（如VS）配置筛选器

使用`source_group`命令将源文件分组即可做到：

- 如果头文件和源文件分开放在不同目录，并且已经按照一定规则组织好了目录，最简单的方式就是筛选器完全按照文件目录来配置：
```CMake
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} [PREFIX "Some Prefix"] FILES ${lib_sources} ${lib_headers})
```
- 如果头文件源文件混在一起，并且按照一定规则组织，那么可以按照目录结构组织筛选器，并区分一下源文件和头文件：
```CMake
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} PREFIX "Header Files" FILES ${lib_headers})
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} PREFIX "Source Files" FILES ${lib_sources})
```
- 如果所有东西全部混在一起，且需要精细的重新组织，那么需要全部手动组织：
```CMake
source_group("Source Files/sys" FILES ${sys_sources})
source_group("Source Files/gl" FILES ${gl_sources})
...
source_group("Header Files/sys" FILES ${sys_headers})
source_group("Header Files/gl" FILES ${gl_headers})
...
```
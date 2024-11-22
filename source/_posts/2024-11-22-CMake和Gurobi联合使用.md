---
title: CMake和Gurobi联合使用
date: 2024-11-22 16:48:46
tags: 
    - Gurobi
    - C++
    - Cmake
---

以往都是用Python调用Gurobi进行数学规划建模，最近开始涉及C++的编程。由于不想使用C++和python联合编程（也许是C++调用一个Python解释器？总之，没这样去做），所以直接用C++调用Gurobi了。
由于我用的是mac，因此`VScode + Cmake`是最佳的组合，本文主要记录的是Cmake文件应该如何编写。(**Windows系统请不要参考这个教程，不能保证正确性**)

首先，这是Gurobi官方的链接：
https://support.gurobi.com/hc/en-us/articles/360039499751-How-do-I-use-CMake-to-build-Gurobi-C-C-projects

按照官方的链接，在Gurobi正确安装的情况下， `mip1_c++.cpp`亲测可以运行。如果发现，上述的教程不能通过Cmake编译，且原因是找不到Gurobi，那大概率是Gurobi的环境变量设置的有问题了，需要修改环境变量到指定的路径，
这里放上macos的`.zshrc`设置。

``` bash
export GUROBI_LICENSE_FILE=/Users/yurunfeng/gurobi.lic
export GUROBI_HOME=/Library/gurobi1103/macos_universal2
```

配置好环境变量，编译应该就没问题了。

在自己的项目中，把`FindGUROBI.cmake`文件直接放在工程目录中， `CmakeLists.txt` **需要自己更改**。`FindGUROBI.cmake`也可根据需要进行更改，
比如以下这个例子：

``` bash
# CmakeLists.txt 
# 这只是一个例子，如果不知道是什么意思的话，不要直接抄，肯定是运行不了的
cmake_minimum_required(VERSION 3.29)
cmake_policy(SET CMP0167 NEW)
enable_language(CXX)

# set(CMAKE_OSX_ARCHITECTURES "arm64")
# set(GUROBI_INCLUDE_DIRS "/Library/gurobi1103/macos_universal2/include")  
# set(GUROBI_CXX_LIBRARY "/Library/gurobi1103/macos_universal2/lib/libgurobi_c++.a") 
set(CMAKE_PREFIX_PATH "/Users/yurunfeng/vcpkg/installed/arm64-osx/share")
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CXXFLAGS -D_GNU_SOURCE True)

project(FWPSP)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

find_package(nlohmann_json CONFIG REQUIRED) # json 
find_package(Boost REQUIRED COMPONENTS program_options stacktrace_basic stacktrace_noop system)
find_package(Boost REQUIRED COMPONENTS log log_setup thread filesystem system date_time regex)
find_package(GUROBI REQUIRED)

include_directories(${GUROBI_INCLUDE_DIRS})
include_directories(${Boost_INCLUDE_DIRS})

set(sources 
  src/main.cpp
  src/alg_config.cpp
  src/log_config.cpp
  src/ip_model.cpp
  src/data_loader.cpp
  src/cbs_node.cpp
  src/cbs.cpp
  src/node_manager.cpp
)

add_executable(${CMAKE_PROJECT_NAME} ${sources})

target_include_directories(FWPSP PRIVATE ${CMAKE_SOURCE_DIR}/inc)
target_link_libraries(${CMAKE_PROJECT_NAME} 
        nlohmann_json::nlohmann_json 
        ${Boost_LIBRARIES}
        ${GUROBI_LIBRARY}
        optimized ${GUROBI_CXX_LIBRARY}
        debug ${GUROBI_CXX_LIBRARY}
)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
  include(FeatureSummary)
  feature_summary(WHAT ALL)
endif()

```

总之，重点在于如何告诉Cmake链接到Gurobi，如果某个环节错了的话，那就检查这个环节的问题，Cmake不熟悉也只能多学习Cmake了（因为我也不熟悉啊🥹）。
此外，如果Gurobi的环境变量没设置也没关系，上述的`CmakeLists.txt`被注释的部分就是设置Gurobi的路径(macos)：


``` bash
set(GUROBI_INCLUDE_DIRS "/Library/gurobi1103/macos_universal2/include")  
set(GUROBI_CXX_LIBRARY "/Library/gurobi1103/macos_universal2/lib/libgurobi_c++.a") 
```
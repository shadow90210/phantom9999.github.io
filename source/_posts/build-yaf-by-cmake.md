---
title: 使用cmake构建yaf框架
categories: php-devel
comments: false
tags:
  - php-devel
  - php
  - cmake
  - yaf
keywords: 'php, cmake, yaf, 编译'
description: 使用cmake构建yaf框架
abbrlink: 4d3b3d24
date: 2018-07-07 16:00:31
updated: 2018-07-07 16:00:31
---

# 简介
本文介绍使用cmake构建yaf框架

# 背景
## yaf
yaf是一个使用c语言实现的php框架, 作为php的拓展加载到php中, 具有较高的性能.
php的拓展使用phpize构建, 这类构建工具不友好, 本文将改用更加友好的cmake工具.



## cmake
cmake是一款较为友好的构建工具, 其拥有自己的配置语法.
cmake工具较为友好的一个方面是, 多种集成开发环境对其支持, 包括:
- CLion
- codeblocks
- 等等

比较重要的一点是, CLion使用cmake构建项目, 所以CLion只对能够使用cmake构建的项目友好.
这也是phpize不太友好的一点.



# 改造
cmake将CMakeLists.txt作为项目管理文件, 改造的过程变为编写CMakeLists.txt的过程.

## 准备
改造之前需要准备的工具包括:
- 安装cmake
- 安装php开发包
- 生成config.h

执行<code>yum install cmake</code>命令完成cmake的安装.
执行<code>yum install php-devel</code>命令完成php开发包的安装

使用phpize进行构建过程中, 构架工具会生成一个config.h的文件, 这个文件定义了一些重要的宏.
为了降低CMakeLists.txt的编写成本, 本文将通过phpize工具生成config.h, 然后直接使用config.h.





## 编写CMakeLists.txt
```
cmake_minimum_required(VERSION 2.8)
project(reading_yaf)

# 寻找php-config目录
if (DEFINED PHP_CONFIG_DIR)
    set(PHP_CONFIG_DIR "${PHP_CONFIG_DIR}/")
else ()
    set(PHP_CONFIG_DIR "")
endif ()

# 读取include目录
execute_process(COMMAND ${PHP_CONFIG_DIR}php-config --include-dir
        OUTPUT_VARIABLE PHP_INCLUDE_DIR
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
# 读取链接库
execute_process(COMMAND ${PHP_CONFIG_DIR}php-config --libs
        OUTPUT_VARIABLE PHP_LIBS
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
# 读取链接参数
execute_process(COMMAND ${PHP_CONFIG_DIR}php-config --ldflags
        OUTPUT_VARIABLE PHP_LDFLAGS
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
# 获取插件存放目录
execute_process(COMMAND ${PHP_CONFIG_DIR}php-config --extension-dir
        #    RESULT_VARIABLE PHP_EXTDIR
        OUTPUT_VARIABLE PHP_EXTDIR
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )

# 添加宏
add_definitions(
        -DZEND_ENABLE_STATIC_TSRMLS_CACHE=1
        -DHAVE_CONFIG_H
        -DPHP_ATOM_INC
)
# 包含目录, 保持与php-config --includes的结果一致
include_directories(
        BEFORE
        ${PHP_INCLUDE_DIR}
        ${PHP_INCLUDE_DIR}/Zend
        ${PHP_INCLUDE_DIR}/main
        ${PHP_INCLUDE_DIR}/TSRM
        ${PHP_INCLUDE_DIR}/ext
        ${PHP_INCLUDE_DIR}/ext/date/lib
)


# 添加include目录
include_directories(.)
include_directories(configs)
include_directories(requests)
include_directories(responses)
include_directories(routes)
include_directories(views)

# 获取源文件
aux_source_directory(. SRC)
aux_source_directory(configs SRC)
aux_source_directory(requests SRC)
aux_source_directory(responses SRC)
aux_source_directory(routes SRC)
aux_source_directory(views SRC)

# 添加构建目标
add_library(yaf SHARED ${SRC})

# 添加安装目录
install(
        TARGETS yaf
        LIBRARY DESTINATION ${PHP_EXTDIR}
)

# 设置构建产物的命名, 包括
# - 去掉"lib"前缀
# - 统一后缀为".so"
set_target_properties(
        yaf PROPERTIES
        PREFIX ""
        SUFFIX ".so"
)
```


# 参考






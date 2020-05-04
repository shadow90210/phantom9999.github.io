---
title: 一些c++库的使用总结
categories: cpp
comments: false
abbrlink: b945c282
date: 2020-05-04 20:06:10
updated: 2020-05-04 20:06:10
tags:
keywords:
description:
---

# 简介

本文介绍一些c++库的使用。

# gflags

## 简单实用

gflags是一个流行的解析命令行的c++库。用户使用这个库定义的变量，可以通过多种途径进行赋值。

gflags支持定义多种数据类型，包括bool、int32、int64、uint64、double、string。

定义一个变量的操作如下：

```
DEFINE_bool(name, "default value", description);
```

然后在main函数开头添加<code>  google::ParseCommandLineFlags(&argc, &argv, true);</code>，在main函数末尾添加<code>  google::ShutDownCommandLineFlags();</code>。

如果需要使用一个已经定义过的flags变量，则需要使用<code>DECLARE_XXX</code>，否则将会提示没有申明过这个变量。

gflags提供参数校验器，使用RegisterFlagValidator函数注册校验器。

gflags提供google::SetVersionString()函数设置版本信息(—version时返回的字符)。

gflags提供gflags::SetUsageMessage函数设置帮助信息（—help时返回的字符）。

gflags提供--flagfile参数指定配置文件，gflags将从这个配置文件中读取配置信息，同时，这个配置文件中也支持—flagfile读取配置。

gflags提供—fromenv参数从环境变量中读取配置信息。

## 编译实用

gflags一个比较坑的地方是，默认情况下gflags库的namespace是flags，而一大批依赖这个库的第三方库使用的google名字空间下的gflags。为了解决这个问题，需要在编译时指定名字空间（添加参数-DGFLAGS_NAMESPACE=google)。

有时候可能把gflags链接到动态链接库中，编译时的flags需要添加-fPIC，解决方法是在gflags中找到CMakeLists.txt中添加add_compile_options(-fPIC) 。

默认情况下gflags使用Release编译的，为了方便调试，建议设置-DCMAKE_BUILD_TYPE=RelWithDebInfo，即设置-o2 -g



# glog

## 使用

又是一个使用方便的日志库，这个库是同步日志的，性能不是特别好，不适合高并发中使用。

使用起来极其方便，包含头文件(glog/logging.h)，然后直接使用。

需要注意的是，FATAL日志会出core。

glog支持DLOG功能，这类日志在添加NDEBUG下不会打印DLOG日志，在没有这个flag下会打印这个日志，方便用户调试。

glog支持VLOG功能，这类日志划分为多个级别，通过命令行参数控制打印的级别。

使用时，在main函数开始添加google::InitGoogleLogging(argv[0]);开启使用glog之旅。

## 编译

glog依赖gflags库，使用glog的话建议先安装一下gflags，同时名字空间设置成google。

编译glog之前，先指定gflags路径，使用参数CMAKE_INCLUDE_PATH和CMAKE_LIBRARY_PATH完成。



# rocksdb

## 使用



## 编译

这个库依赖flags，所以编译前可以指定一下gflags，并添加参数-DWITH_GFLAGS=1指定依赖flags。



# fruit

## 使用





## 编译

这个库依赖boost，编译时需要指定boost的目录，使用参数-DBoost_INCLUDE_DIR=指定boost位置。

这个库编译时建议添加-DBUILD_SHARED_LIBS=OFF关闭动态链接库。




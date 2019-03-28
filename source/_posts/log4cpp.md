---
title: 日志代码收敛(log4cpp/glog)
categories: essay
comments: false
abbrlink: b270d3a9
date: 2018-08-12 18:18:58
updated: 2018-08-12 18:18:58
tags:
keywords:
description:
---

# 简介

本文介绍某项目中日志代码的收敛工作.
这个项目由多个模块组成, 各个模块使用的日志系统不同, 有的模块使用log4cpp, 有的模块使用glog.
这给项目的管理带来了一定的困难, 因此需要对整个项目进行改造, 统一日志系统.

# 背景
## 日志库
目前c++常用并且开源的日志库包括:

- log4cxx
- log4cpp
- log4cplus
- glog
- g3log
- boost.log
- boost.log v2


### log4cxx
log4cxx是apache的log4j的官方c++实现, 架构类似于log4j, 使用跟log4j兼容的配置文件.
这个项目已经不再更新.
在centos的软件库中包含了log4cxx的开发包(log4cxx-devel).

### log4cpp
log4cpp是log4j的一个非官方实现, 与log4j类似的架构, 并且兼容log4j的配置文件.
这个项目最新更新的时间是2017, 并且已经一年没更新了.
在centos的软件库中包含了log4cpp的开发包(log4cpp-devel).


### log4cplus
log4cplus是log4j的另一个非官方实现, 与log4j类似的架构, 并且兼容log4j的配置文件.
这个项目一直在更新, 并且已经发布2.0版本.
在centos的软件库中包含了log4cplus的开发包(log4cplus)

### glog
glog是谷歌开源的日志系统, 以简单著称, 支持的功能包括:

1， 参数设置，以命令行参数的方式设置标志参数来控制日志记录行为；
2， 严重性分级，根据日志严重性分级记录日志；
3， 可有条件地记录日志信息；
4， 条件中止程序。丰富的条件判定宏，可预设程序终止条件；
5， 异常信号处理。程序异常情况，可自定义异常处理过程；
6， 支持debug功能。可只用于debug模式；
7， 自定义日志信息；
8， 线程安全日志记录方式；
9， 系统级日志记录；
10， google perror风格日志信息；
11， 精简日志字符串信息。


### g3log
glog不支持异步日志, 并且性能较差. 于是有开发者基于glog开发了支持异步日志的新日志系统, 命名为g3log.


### boost.log v1
这个日志系统出现较早, 但是一直没有合入到boost套装中.
用户如果要使用这个库, 则需要单独编译.


### boost.log v2
从boost 1.54开始, boost.log v2加入boost库, 但是目前centos7的boost版本是1.53.
boost将日志系统进行分层, 类似于log4j, 并且同时支持同步和异步日志.


## 项目
目前项目的A模块使用log4cpp作为日志


























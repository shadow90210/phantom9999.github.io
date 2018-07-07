---
title: yaf源码阅读
categories: essay
comments: false
tags:
  - yaf
  - php
keywords: 'php, yaf, 源码阅读'
description: 本文介绍yaf的源码解读
abbrlink: c1026c5e
date: 2018-07-07 19:33:31
updated: 2018-07-07 19:33:31
---


# 简介

本文将分别从接口层和实现层解读yaf框架.

# 接口层

## 介绍

yaf是一个使用c实现的高性能框架. 它以php拓展的形式实现整个php框架.

它的优点包括:
1. 用C语言开发的PHP框架, 相比原生的PHP, 几乎不会带来额外的性能开销.
2. 所有的框架类, 不需要编译, 在PHP启动的时候加载, 并常驻内存.
3. 更短的内存周转周期, 提高内存利用率, 降低内存占用率.
4. 灵巧的自动加载. 支持全局和局部两种加载规则, 方便类库共享.
5. 高性能的视图引擎.
6. 高度灵活可扩展的框架, 支持自定义视图引擎, 支持插件, 支持自定义路由等等.
7. 内建多种路由, 可以兼容目前常见的各种路由协议.
8. 强大而又高度灵活的配置文件支持. 并支持缓存配置文件, 避免复杂的配置结构带来的性能损失.
9. 在框架本身,对危险的操作习惯做了禁止.
10. 更快的执行速度, 更少的内存占用.

## 框架执行

整个框架分为应用程序组件集和其他基础组件集. 在应用程序组件集中包含了, 插件, 分发器, 路由器, 控制器, 启动器等. 基础组件包括, session组件, 注册表组件, 自动加载组件, 配置组件.

yaf框架包含多个层次, 分别是:

- Application 应用程序, 一个服务作为一个应用程序
- Module 模块, 一个服务包含多个模块
- Controller 控制器, 一个模块包含多个控制器
- Action 动作, 一个控制器包含多个动作

yaf是传统的php框架, 使用传统的执行流程. 这类执行流程无法上下文复用, 每处理一个请求就要初始化一次框架.

处理用户请求时, yaf框架先创建应用程序(Application类对象), 应用程序类创建启动器(Bootstrap类对象), 启动器执行分发器加载, 插件加载等操作. 然后应用程序将请求封装成请求对象(Request类对象), 将请求对象交付给分发器, 分发器根据路由器中的路由规则, 获得对应的处理对象. 路由器根据URI信息, 查询路由规则, 找到匹配的路由规则, 根据路由规则获得相应的处理器(Controller类对象), 并将其返回给分发器. 分发器执行处理器, 将处理结果封装成相应对象(Response对象)返回给应用程序. 应用程序将响应对象返回给用户. 框架的执行流程如图所示:

![yaf执行流程图](/images/yaf_sequence.png)



## 自动加载组件

yaf框架为了兼容老版本的php, 提供两种自动加载的策略, 分别是基于下划线的自动加载方案和名字空间加载方案. yaf的加载方案在配置文件中指定.

yaf中库的分为本地库和全局库, 全局库作用于整个应用程序, 本地库只作用于当前模块(Module), 当当前应用程序中只包含一个模块时, 全局库和本地库的区别只是位置的不同罢了. 

### 下划线自动加载方案

这种自动加载方案会对需要加载的类的类名进行处理, 将类名中的下划线(<code>_</code>)替换为替换为斜线(<code>/</code>), 生成类路径, 然后进行库文件查找. 库查找时, 先获得本地库的路径, 接着融合本地库路径和类目录, 得到类文件的绝对路径, 然后判断这个文件是否存在, 如果存在, 则进行加载, 否则进行全局库查找.

执行的流程如下:

```flow
begin=>start: 开始
end=>end: 结束
get_class_name=>operation: 获得类名
process=>operation: 下划线替换为斜线,生成类路径
get_global_lib=>operation: 获得全局库绝对路径
get_local_lib=>operation: 获得本地库路径
merge_global=>operation: 合并全局库路径和类路径, 生成文件绝对路径
merge_local=>operation: 合并本地库路径和类路径, 生成文件绝对路径
judge_global=>condition: 判断文件是否存在
judge_local=>condition: 判断文件是否存在
load_error=>operation: 加载失败
load_global=>operation: 加载文件
load_local=>operation: 加载文件

begin->get_class_name->process->get_local_lib->merge_local->judge_local
judge_local(yes)->load_local->end
judge_local(no)->get_global_lib->merge_global->judge_global
judge_global(yes)->load_global->end
judge_global(no)->load_error->end
```







## 路由组件






## 分发器组件





## 插件机制

从图中可以看到, 整个框架的核心是这个分发器. yaf框架针对分发的执行流程, 在各个执行片段添加了钩子, 并通过这些钩子制作了插件机制.

yaf的钩子包括:

- routerStartup 在查询路由规则之前执行
- routerShutdown 在查询完路由规则之前执行
- dispatchLoopStart 在分发流程开始前执行
- preDispatch 在分发操作之前执行
- postDispatch 在分发操作完毕执行
- dispatchLoopShutdown 在分发流程结束后执行


# 实现层



# 参考
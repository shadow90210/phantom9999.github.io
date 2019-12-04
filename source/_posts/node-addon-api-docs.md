---
title: node-addon-api使用文档
categories: essay
comments: false
abbrlink: 10f6a378
date: 2019-03-29 22:34:39
updated: 2019-03-29 22:34:39
tags:
keywords:
description:
---
# 简介
"node-addon-api"是nodejs的"n-api"接口的c++封装, 通过提供c++对象模型和异常异常处理方式, 简化nodejs开发的成本.

"n-api"是nodejs为原生拓展提供的c语言风格ABI, 它独立于js运行环境, 旨在屏蔽js运行环境的差异, 让拓展能够运行在不同版本的nodejs下.

"node-addon-api"不作为nodejs的组件发布, 它只是基于nodejs的"n-api", 这样, "node-addon-api"将于nodejs本身解耦, 基于"node-addon-api"的拓展运行在新nodejs环境时, 不需要重新编译.

值得注意的是, nodejs的其他接口, 例如<code>libuv</code>等, 并不包含在nodejs的拓展ABI内, 基于这些接口开发的nodejs拓展无法保证在多个版本的nodejs环境中运行. 由于"n-api"是nodejs 6.x之后才出现的, 因此这套ABI只支持nodejs6.x之后的版本. 

当新的api被添加到n-api后, node-addon-api必须马上更新, 否则node-addon-api就无法使用这部分特性.

本文将数据结构, 异常处理, 与js交互等多个方面介绍node-addon-api.

# 数据结构
首先是node-addon-api的数据结构, node-addon-api的数据结构是封装的n-api的, 我把它的数据结构分为两类, 一类是js中有直接对应类型的, 另一类是js中没有直接对应的类型, 但node-addon-api中出现的.

## 基本类型



| js类型      | c++类       | 说明                               |
| ----------- | ----------- | ---------------------------------- |
| string      | String      | 字符串, 使用unicode存储            |
| number      | Number      | 数字, 使用浮点存储                 |
| boolean     | Boolean     | 布尔值                             |
| bigInt      | BigInt      | 大整形, 新引入, 使用uint64数组存储 |
| object      | Object      | 对象                               |
| symbol      | Symbol      | 符号                               |
| Buffer      | Buffer      | 二进制类型, 不受gc管理             |
| function    | Function    | 函数类型                           |
| arrayBuffer | ArrayBuffer | buffer数组类型                     |
| typedArray  | TypedArray  | 类型数组                           |
| DataView    | DataView    | 视图                               |
| promises    | Promises    |                                    |



## 特殊类型

| 特殊类型                | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| Name                    | 特殊的类型, 可以用来作为类的属性名, 支持字符串和symbol       |
| Env                     | 特殊结构, 包含了当前请求的运行环境                           |
| Value                   | js类型的c++表现类型, js类型的基类                            |
| CallbackInfo            | 特殊结构, 包含函数参数列表和Env对象, 常出现在c++实现的js函数参数表中, , 由nodejs环境生成传递给自定义函数. |
| Reference               | 特殊引用类型, 类似于c++中的shared_ptr, 创建时不添加计数器. 当计数器为0时, 不负责删除数据, 使用垃圾回收机制回收数据. |
| External                | 用来包装c++数据的结构, 方便用户管理自定义结构, 这个类提供自定义清理函数接口, 可以让用户制定清理方法. |
| ObjectReference         | 对象引用类型, 是Reference的子类, 包含引用对象和一个计数器, 相当于shared_ptr. |
| PropertyDescriptor      | js对象的属性描述, 可以是函数/变量/访问控制等.                |
| FunctionReference       |                                                              |
| ObjectWrap              |                                                              |
| ClassPropertyDescriptor |                                                              |
|                         |                                                              |
|                         |                                                              |








## 对象及引用
# 与js交互
## 自定义函数
## 自定义类
## js函数调用
## 异步操作
## 异常处理
# 内存管理
## 生命周期
## 额外类型
# 其他
## 版本管理










# 异常处理


# 内存管理
## 对象生命周期管理
# 异步操作
# promises
# 版本管理









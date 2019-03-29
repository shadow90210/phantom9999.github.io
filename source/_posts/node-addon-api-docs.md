---
title: node-addon-api-docs
categories: essay
comments: false
date: 2019-03-29 22:34:39
updated: 2019-03-29 22:34:39
tags:
keywords:
description:
---

[TOC]

# 基本类型
Node Addon API consists of a few fundamental data types. These allow a user of the API to create, convert and introspect fundamental JavaScript types, and interoperate with their C++ counterparts.

| 基本类型                                                     | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Array](https://github.com/nodejs/node-addon-api/blob/master/doc/basic_types.md#array) | 对应js中的数组                                               |
| [Symbol](https://github.com/nodejs/node-addon-api/blob/master/doc/symbol.md) | 对应js中的symbol                                             |
| [String](https://github.com/nodejs/node-addon-api/blob/master/doc/string.md) | 对应js中的字符串                                             |
| [Name](https://github.com/nodejs/node-addon-api/blob/master/doc/basic_types.md#name) | 特殊的类型, 可以用来作为类的属性名, 支持字符串和symbol       |
| [Number](https://github.com/nodejs/node-addon-api/blob/master/doc/number.md) | 对应js中的number                                             |
| [BigInt](https://github.com/nodejs/node-addon-api/blob/master/doc/bigint.md) | 对应js中的bigInt类型, 底层使用uint64数组存储.                |
| [Boolean](https://github.com/nodejs/node-addon-api/blob/master/doc/boolean.md) | 对应js中的布尔类型                                           |
| [Env](https://github.com/nodejs/node-addon-api/blob/master/doc/env.md) | 特殊结构, 包含了当前请求的运行环境                           |
| [Value](https://github.com/nodejs/node-addon-api/blob/master/doc/value.md) | js类型的c++表现类型, js类型的基类                            |
| [CallbackInfo](https://github.com/nodejs/node-addon-api/blob/master/doc/callbackinfo.md) | 特殊结构, 包含函数参数列表和Env对象, 常出现在c++实现的js函数参数表中, , 由nodejs环境生成传递给自定义函数. |
| [Reference](https://github.com/nodejs/node-addon-api/blob/master/doc/reference.md) | 特殊引用类型, 类似于c++中的shared_ptr, 创建时不添加计数器. 当计数器为0时, 不负责删除数据, 使用垃圾回收机制回收数据. |
| [External](https://github.com/nodejs/node-addon-api/blob/master/doc/external.md) | 用来包装c++数据的结构, 方便用户管理自定义结构, 这个类提供自定义清理函数接口, 可以让用户制定清理方法. |
| [Object](https://github.com/nodejs/node-addon-api/blob/master/doc/object.md) | 对应js的对象, 提供了多个转个函数                             |
| [ObjectReference](https://github.com/nodejs/node-addon-api/blob/master/doc/object_reference.md) | 对象引用类型, 是Reference的子类, 包含引用对象和一个计数器, 相当于shared_ptr. |
| [PropertyDescriptor](https://github.com/nodejs/node-addon-api/blob/master/doc/property_descriptor.md) | js对象的属性描述, 可以是函数/变量/访问控制等.                |



# 异常处理
## Error
## TypeError
## RangeError
# Object Lifetime Management
## HandleScope
## EscapableHandleScope
# Working with JavaScript Values
## Function
### FunctionReference
## ObjectWrap
### ClassPropertyDescriptor
## Buffer
## ArrayBuffer
## TypedArray
### TypedArrayOf
## DataView
# Memory Management
# Async Operations
## AsyncWorker
## AsyncContext
# Promises
# Version management



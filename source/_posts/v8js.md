---
title: v8嵌入式开发--v8js篇
categories: essay
comments: false
abbrlink: 137637df
date: 2019-12-04 22:40:00
updated: 2019-12-04 22:40:00
tags:
  - v8
  - js
  - javascript
  - nodejs
keywords: 'v8, js, javascript, nodejs'
description:
---

# 简介

v8js是一个特殊的php拓展, 其作用是将v8嵌入到php中, 使得用户可以在php中运行js代码. 同时, 经过作者的努力, 运行在php中的js可以无缝访问并php中的数据结构, 调用php內建的函数, 从而实现"1 + 1 > 2"的目标. 本文将跟随作者的, 领略v8js的风采.

# 功能介绍



使用一个工具是了解这个工具的最好方式, 笔者将在这一节中介绍v8js拓展的功能及使用.

首先介绍一下v8js的提供的接口, v8js的接口如下:



```php
<?php
class V8Js
{
    /* Constants */

    const V8_VERSION = '';

    const FLAG_NONE = 1;
    const FLAG_FORCE_ARRAY = 2;
    const FLAG_PROPAGATE_PHP_EXCEPTIONS = 4;

    /* Methods */

    /**
     * Initializes and starts V8 engine and returns new V8Js object with it's own V8 context.
     * @param string $object_name
     * @param array $variables
     * @param array $extensions
     * @param bool $report_uncaught_exceptions
     * @param string $snapshot_blob
     */
    public function __construct($object_name = "PHP", array $variables = [], array $extensions = [], $report_uncaught_exceptions = TRUE, $snapshot_blob = NULL)
    {}

    /**
     * Provide a function or method to be used to load required modules. This can be any valid PHP callable.
     * The loader function will receive the normalised module path and should return Javascript code to be executed.
     * @param callable $loader
     */
    public function setModuleLoader(callable $loader)
    {}

    /**
     * Provide a function or method to be used to normalise module paths. This can be any valid PHP callable.
     * This can be used in combination with setModuleLoader to influence normalisation of the module path (which
     * is normally done by V8Js itself but can be overriden this way).
     * The normaliser function will receive the base path of the current module (if any; otherwise an empty string)
     * and the literate string provided to the require method and should return an array of two strings (the new
     * module base path as well as the normalised name).  Both are joined by a '/' and then passed on to the
     * module loader (unless the module was cached before).
     * @param callable $normaliser
     */
    public function setModuleNormaliser(callable $normaliser)
    {}

    /**
     * Compiles and executes script in object's context with optional identifier string.
     * A time limit (milliseconds) and/or memory limit (bytes) can be provided to restrict execution. These options will throw a V8JsTimeLimitException or V8JsMemoryLimitException.
     * @param string $script
     * @param string $identifier
     * @param int $flags
     * @param int $time_limit in milliseconds
     * @param int $memory_limit in bytes
     * @return mixed
     */
    public function executeString($script, $identifier = '', $flags = V8Js::FLAG_NONE, $time_limit = 0, $memory_limit = 0)
    {}

    /**
     * Compiles a script in object's context with optional identifier string.
     * @param $script
     * @param string $identifier
     * @return resource
     */
    public function compileString($script, $identifier = '')
    {}

    /**
     * Executes a precompiled script in object's context.
     * A time limit (milliseconds) and/or memory limit (bytes) can be provided to restrict execution. These options will throw a V8JsTimeLimitException or V8JsMemoryLimitException.
     * @param resource $script
     * @param int $flags
     * @param int $time_limit
     * @param int $memory_limit
     */
    public function executeScript($script, $flags = V8Js::FLAG_NONE, $time_limit = 0 , $memory_limit = 0)
    {}

    /**
     * Set the time limit (in milliseconds) for this V8Js object
     * works similar to the set_time_limit php
     * @param int $limit
     */
    public function setTimeLimit($limit)
    {}

    /**
     * Set the memory limit (in bytes) for this V8Js object
     * @param int $limit
     */
    public function setMemoryLimit($limit)
    {}

    /**
     * Set the average object size (in bytes) for this V8Js object.
     * V8's "amount of external memory" is adjusted by this value for every exported object.  V8 triggers a garbage collection once this totals to 192 MB.
     * @param int $average_object_size
     */
    public function setAverageObjectSize($average_object_size)
    {}

    /**
     * Returns uncaught pending exception or null if there is no pending exception.
     * @return V8JsScriptException|null
     */
    public function getPendingException()
    {}

    /**
     * Clears the uncaught pending exception
     */
    public function clearPendingException()
    {}

    /** Static methods **/

    /**
     * Registers persistent context independent global Javascript extension.
     * NOTE! These extensions exist until PHP is shutdown and they need to be registered before V8 is initialized.
     * For best performance V8 is initialized only once per process thus this call has to be done before any V8Js objects are created!
     * @param string $extension_name
     * @param string $code
     * @param array $dependencies
     * @param bool $auto_enable
     * @return bool
     */
    public static function registerExtension($extension_name, $code, array $dependencies, $auto_enable = FALSE)
    {}

    /**
     * Returns extensions successfully registered with V8Js::registerExtension().
     * @return array|string[]
     */
    public static function getExtensions()
    {}

    /**
     * Creates a custom V8 heap snapshot with the provided JavaScript source embedded.
     * Snapshots are supported by V8 4.3.7 and higher.  For older versions of V8 this
     * extension doesn't provide this method.
     * @param string $embed_source
     * @return string|false
     */
    public static function createSnapshot($embed_source)
    {}
}

final class V8JsScriptException extends Exception
{
    /**
     * @return string
     */
    final public function getJsFileName( ) {}

    /**
     * @return int
     */
    final public function getJsLineNumber( ) {}
    /**
     * @return int
     */
    final public function getJsStartColumn( ) {}
    /**
     * @return int
     */
    final public function getJsEndColumn( ) {}

    /**
     * @return string
     */
    final public function getJsSourceLine( ) {}
    /**
     * @return string
     */
    final public function getJsTrace( ) {}
}

final class V8JsTimeLimitException extends Exception
{
}

final class V8JsMemoryLimitException extends Exception
{
}


```



从上面接口的注释可以知道, 使用v8js拓展执行js代码时, 基本流程是先创建一个V8Js类的对象, 然后执行成员函数executeString, 函数返回的结果即是js的执行结果.

V8js的成员函数<code>setTimeLimit</code>和<code>setMemoryLimit</code>函数用于设置js执行的时间限制和内存限制.

使用例子后续补充.




# 架构设计
接着讲一讲v8js拓展的架构设计. v8js模块包含了一个计时器线程, 计时任务双线队列, 全局变量结构体. 在全局变量结构体中存储了v8的platform. v8js模块中实现了一个名为V8Js的类, 在这个类的构造函数中, v8js会出现一个isolate对象, 并创建一个全局上下文. 那么v8js模块中, 一个V8Js类的对象包含一个独立的isolate对象. 整体架构如下图所示:





# 详细设计
## js代码执行
v8的源码中提供了一个hello的代码, 如下所示:

```c++
// 初始化周边数据
v8::V8::InitializeICUDefaultLocation(argv[0]);
v8::V8::InitializeExternalStartupData(argv[0]);

// 创建platform
std::unique_ptr<v8::Platform> platform = v8::platform::NewDefaultPlatform();

// 初始化V8环境
v8::V8::InitializePlatform(platform.get());
v8::V8::Initialize();

// 构造参数
v8::Isolate::CreateParams create_params;
create_params.array_buffer_allocator = v8::ArrayBuffer::Allocator::NewDefaultAllocator();

// 创建isolate对象(v8虚拟机实例)
v8::Isolate* isolate = v8::Isolate::New(create_params);
v8::Isolate::Scope isolate_scope(isolate);
v8::HandleScope handle_scope(isolate);

// 创建上下文
v8::Local<v8::Context> context = v8::Context::New(isolate);
v8::Context::Scope context_scope(context);

// 将普通字符串转化为V8的字符串
v8::Local<v8::String> source = v8::String::NewFromUtf8(isolate, "'Hello' + ', World!'",  v8::NewStringType::kNormal).ToLocalChecked();

// 编译
v8::Local<v8::Script> script = v8::Script::Compile(context, source).ToLocalChecked();

// 执行
v8::Local<v8::Value> result = script->Run(context).ToLocalChecked();

// 执行结果转化为普通字符串
v8::String::Utf8Value utf8(isolate, result);
printf("%s\n", *utf8);

// 销毁isolate
isolate->Dispose();

// 销毁V8环境和platform
v8::V8::Dispose();
v8::V8::ShutdownPlatform();
delete create_params.array_buffer_allocator;
```
从demo可以看到, 运行一个js脚本需要的操作包括:
- 创建platform
- 创建isolate
- 创建context
- 字符串编译成script
- 运行script
- 取出结果
- 销毁环境

v8js拓展将platform创建放在了第一个V8Js类的构造函数中; isolate和context的创建放在了V8Js的构造函数. executeString中执行了编译和运行script的操作. 对象析构的时候销毁isolate对象. 模块退出的时候销毁platform等操作.


## 內建函数实现
在这个部分需要介绍一下v8的template机制以及v8各个元素之间的关系. 下面是v8的常用名词:

- platform 平台, 一个进程中可以有多个, 但是只有一个生效
- isolate v8虚拟机, 可以存在多个, 一个isolate同一时间只允许一个线程访问
- context 运行上下文, 可以有多个, 可以嵌套, 从属于isolate. 
- scope 作用域, isolate和context都有scope, 用于垃圾回收处理
- template 模板, 用于创建函数和对象, 从属于isolate

在v8中, 对象和函数从属于context, 而context在isolate的scope销毁时会被一同销毁, 那么这些对象和函数需要在context创建的时候被不停的创建, 为了省去这部分工作量, v8引入了template, 用于创建对象和函数.

內建函数的实现就是基于template实现的. 首先V8Js的的构造函数中会创建一个template对象, 然后再这个对象中添加functionTemplate对象, 而这些functionTemplate对象封装了內建函数. 在contenxt创建的时候, 将这个template对象作为实参传入, 那么创建出来的context就包含了內建函数.

template包含ObjectTemplate和FunctionTemplate两种, 前者创建对象, 后者创建函数. 如果用户想创建一个类呢? 由于js早期并不存在class关键词, 创建一个类的对象都是通过函数实现的. 所以创建一个类需要使用的是FunctionTemplate.

v8js拓展的內建函数的具体实现在文件"v8js_methods.cc"中, 在<code>v8js_register_methods</code>函数中将这些內建函数添加到template对象中. 这个函数在V8Js类的构造函数中被调用, 位置在contenxt被创建之前.




## commonjs模块实现
首先讲一下什么是commonjs模块规范. "规范"认为, 每个文件是一个模块(module), 并拥有其自己的作用域. 在这个作用域内声明的变量和函数都是这个作用域私有的. 模块信息存储在module对象中, module提供exports对象用于暴露作用域内的函数/变量. 模块之间通过require函数加载其他的模块.

为了实现上面规范, 在每个文件的作用域内, 需要提前声明module, exports对象和require函数. 并且module和exports对象存在于模块中, 模块间的module和exports都不同. 参考上文中的template的功能, module和exports可以实现为两个ObjectTemplate, 并注册到主ObjectTemplate. 同样, require也可以类似实现. 这时基于主ObjectTemplate创建context, 并在这个context运行指定文件, 并提取exports, 即可暴露指定的接口(变量和函数).

考虑到重新创建context成本较大, v8的context支持基于老context创建新的context, 但是基于这种方式就无法使用到template了. nodejs的解决方案是, 创建一个函数, 在函数体中添加模块的代码, 将module/exports/require作为形参传入到. 例如, 模块的代码如下:

```
var x = 5;
var addX = function (value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

nodejs编译编译这个代码之前, 将这个模块的代码处理为:

```
(function(module, exports, require) {
    var x = 5;
    var addX = function (value) {
        return value + x;
    };
    module.exports.x = x;
    module.exports.addX = addX;
});
```
然后, 创建module对象, exports对象, reuqire函数, 将这三个作为实参传入到生成的函数中去, 最后提取module和exports对象缓存起来. 当其他的模块reuqire这个模块时, 先查缓存, 命中缓存则直接返回exports对象.
v8js借鉴了nodejs的方式实现了一个commonjs模块.
由于reuqire一个模块时, 可以指定模块的相对路径, 这就要求reuqire在被调用时能够知道调用reuqire的文件的绝对路径. v8创建函数时支持给函数传递一个meta数据, v8js利用这个特性给require传递了路径数据. "v8js_commonjs.cc"中实现了一套相对路径推算绝对路径的代码, 使用c++17或者boost的话, 可以使用filesystem的canonical函数解决.


## 超限功能实现
前文提到v8js拓展包含一个计时器线程和计时任务队列, v8js依靠它们实现了超限功能.
V8Js类的对象在执行js脚本时, v8js会基于isolate/context/超时配置/内存限制创建一个超限任务, 并添加到计时任务队列中. 
计时器线程会取出队列中的任务, 检查是否超时, 内存是否超限. 对超限的任务, 超时线程将终止任务运行. 在判断内存超限时, 计时器线程需要先解锁isolate, 然后获取堆统计信息, 如果超限就手工gc并终止当前任务.




## js中调用php数据实现
这部分略



# 改进与优化
## isolate归属问题
在v8js中isolate是V8Js对象级别的, 事实上isolate和全局context的创建成本较大, 并不适合放在对象级别. 基于php单线程运行的特点, isolate和全局context可以存放在线程级别中. 每次执行脚本时,  基于全局context创建新的context.





# 学习总结

- 学习了js脚本运行的流程
- 学习了v8中各个名词之间的关系
- 学习了多isolate的使用












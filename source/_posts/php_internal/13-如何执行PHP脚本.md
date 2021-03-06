---
title: 13-如何执行PHP脚本
tags: php_internal
categories: php_internal
abbrlink: ffdd5200
date: 2018-01-01 20:07:13
updated: 2018-01-01 20:07:13
---

# 13-如何执行PHP脚本
前面介绍了PHP的生命周期，PHP的SAPI，SAPI处于PHP整个架构较上层，而真正脚本的执行主要由Zend引擎来完成， 这一小节我们介绍PHP脚本的执行。

目前编程语言可以分为两大类：

- 第一类是像C/C++， .NET， Java之类的编译型语言， 它们的共性是：运行之前必须对源代码进行编译，然后运行编译后的目标文件。
- 第二类比如PHP， Javascript， Ruby， Python这些解释型语言， 他们都无需经过编译即可“运行”。

虽然可以理解为直接运行，但它们并不是真的直接就被能被机器理解， 机器只能理解机器语言，那这些语言是怎么被执行的呢， 一般这些语言都需要一个解释器， 由解释器来执行这些源码， 实际上这些语言还是会经过编译环节，只不过它们一般会在运行的时候实时进行编译。为了效率，并不是所有语言在每次执行的时候都会重新编译一遍， 比如PHP的各种opcode缓存扩展(如APC， xcache， eAccelerator等)，比如Python会将编译的中间文件保存成pyc/pyo文件， 避免每次运行重新进行编译所带来的性能损失。

PHP的脚本的执行也需要一个解释器， 比如命令行下的php程序，或者apache的mod_php模块等等。 前面提到了PHP的SAPI接口， 下面就以PHP命令行程序为例解释PHP脚本是怎么被执行的。 例如如下的这段PHP脚本：

    <?php
    $str = "Hello, nowamagic!\n";
    echo $str;
    ?>

假设上面的代码保存在名为hello.php的文件中， 用PHP命令行程序执行这个脚本：

    $ php ./hello.php

这段代码的输出显然是Hello， nowamagic!， 那么在执行脚本的时候PHP/Zend都做了些什么呢？ 这些语句是怎么样让php输出这段话的呢? 下面将一步一步的进行介绍。
程序的执行

1. 如上例中， 传递给php程序需要执行的文件， php程序完成基本的准备工作后启动PHP及Zend引擎， 加载注册的扩展模块。
2. 初始化完成后读取脚本文件，Zend引擎对脚本文件进行词法分析，语法分析。然后编译成opcode执行。 如过安装了apc之类的opcode缓存， 编译环节可能会被跳过而直接从缓存中读取opcode执行。

PHP在读取到脚本文件后首先对代码进行词法分析，PHP的词法分析器是通过lex生成的， 词法规则文件在$PHP_SRC/Zend/zend_language_scanner.l， 这一阶段lex会会将源代码按照词法规则切分一个一个的标记(token)。PHP中提供了一个函数token_get_all()， 该函数接收一个字符串参数， 返回一个按照词法规则切分好的数组。 例如将上面的php代码作为参数传递给这个函数：

    <?php
    $code =<<<PHP_CODE
    <?php
    $str = "Hello, nowamagic\n";
    echo $str;
    PHP_CODE;

    var_dump(token_get_all($code));
    ?>

运行上面的脚本你将会看到一如下的输出：

    array (
      0 =>
      array (
        0 => 368,       // 脚本开始标记
        1 => '<?php     // 匹配到的字符串
    ',
        2 => 1,
      ),
      1 =>
      array (
        0 => 371,
        1 => ' ',
        2 => 2,
      ),
      2 => '=',
      3 =>
      array (
        0 => 371,
        1 => ' ',
        2 => 2,
      ),
      4 =>
      array (
        0 => 315,
        1 => '"Hello, nowamagic
    "',
        2 => 2,
      ),
      5 => ';',
      6 =>
      array (
        0 => 371,
        1 => '
    ',
        2 => 3,
      ),
      7 =>
      array (
        0 => 316,
        1 => 'echo',
        2 => 4,
      ),
      8 =>
      array (
        0 => 371,
        1 => ' ',
        2 => 4,
      ),
      9 => ';',

这也是Zend引擎词法分析做的事情，将代码切分为一个个的标记，然后使用语法分析器(PHP使用bison生成语法分析器， 规则见$PHP_SRC/Zend/zend_language_parser。y)， bison根据规则进行相应的处理， 如果代码找不到匹配的规则，也就是语法错误时Zend引擎会停止，并输出错误信息。 比如缺少括号，或者不符合语法规则的情况都会在这个环节检查。 在匹配到相应的语法规则后，Zend引擎还会进行编译， 将代码编译为opcode， 完成后，Zend引擎会执行这些opcode， 在执行opcode的过程中还有可能会继续重复进行编译-执行， 例如执行eval，include/require等语句， 因为这些语句还会包含或者执行其他文件或者字符串中的脚本。

例如上例中的echo语句会编译为一条ZEND_ECHO指令， 执行过程中，该指令由C函数zend_print_variable(zval* z)执行，将传递进来的字符串打印出来。 为了方便理解， 本例中省去了一些细节，例如opcode指令和处理函数之间的映射关系等。 后面的章节将会详细介绍。

如果想直接查看生成的Opcode，可以使用php的vld扩展查看。扩展下载地址: http://pecl.php.net/package/vld。Win下需要自己编译生成dll文件。

有关PHP脚本编译执行的细节，请阅读后面有关词法分析，语法分析及opcode编译相关内容。

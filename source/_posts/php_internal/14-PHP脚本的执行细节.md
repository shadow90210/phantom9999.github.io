---
title: 14-PHP脚本的执行细节
tags: php_internal
categories: php
---

# 14-PHP脚本的执行细节
众所周知，计算机的CPU只能执行二进制的机器码，每种CPU都有对应的汇编语言，汇编语言编译器将汇编语言翻译成二进制的机器语言，然后CPU开始执行这些机器码。汇编语言作为机器语言与程序设计者之间的一个层，给我们带来了很多方便，程序员不需要用晦涩的01数字来书写程序，当然人们并不满足这样的一个进步，于是在汇编语言之上又多了一个层——C语言，C语言更贴近人类熟悉的“自然语言”，程序设计者可以通过C语言编译器将C源代码文件编译成目标文件（二进制文件，中间会先翻译成汇编语言，然后由汇编语言生成机器码），然后将各个目标文件连接在一起就组成了一个可执行文件。正如有人说过的一句名言“计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决”（“Any problem in computer science can be solved by another layer of indirection.”） PHP语言就是在C语言之上的一个层，PHP引擎是由C语言来实现的，因此PHP语言这一个在C之上抽象出来的层使用起来比C更简单方便，入门门槛更低。

那么，PHP语言究竟如何被执行呢？

PHP语言到C语言之间的转换如果使用“翻译”这个词是不够准确的，因为引擎不是将PHP语言转换成C语言，然后将转换后的C语言编译链接执行。引擎在解析PHP代码的时候通常是分为两个部分，编译和执行：

- 编译阶段：引擎把PHP代码转换成op code中间代码
- 执行阶段：引擎解释并执行编译阶段产生的op code

关于op code会有专门的文章来介绍，现在网络上也已经有很多相关内容的文章，总之PHP代码会被编译成_zend_op_array的形式，这是一个结构体，其中包括很多相关属性，以及最重要的成员zend_op *opcodes，即opcode的数组。执行阶段引擎会按照顺序执行各个opcode。

目前5.3.2版本的PHP中，opcode一共有154种，可以在{PHPSRC}/Zend/zend_vm_opcodes.h看到这些opcode的宏定义。op的结构定义为：

    struct _zend_op {
    	opcode_handler_t handler;
    	znode result;
    	znode op1;
    	znode op2;
    	ulong extended_value;
    	uint lineno;
    	zend_uchar opcode;
    };

其中的成员opcode就对应154个opcode宏定义中的一个，每一个op根据opcode和操作数的类型不同都会对应一个相关的执行句柄(opcode_handler_t handler)，执行句柄是一个函数指针，op的执行执行句柄都定义在{PHPSRC}/Zend/zend_vm_execute.h中，这个文件可以通过一个PHP脚本({PHPSRC}/Zend/zend_vm_gen.php)来生成，这个PHP脚本用来生成zend_vm_opcodes.h和zend_vm_execute.h两个文件，zend_vm_execute.h的内容会根据生成时的参数不同而不同，这里主要是可以定置zend 引擎对op的分发方式，比如用CALL,SWITCH,GOTO,默认的是用CALL，也就是函数调用，所以这里就以函数调用来简单的介绍下这个文件的功能（文件极大，有近36000行，所以不要仔细啃），在这个文件中所有定义为 static int ZEND_FASTCALL 并且以 ZEND_* 开头的函数就是op的句柄，此文件中第一个函数execute是执行op的主方法，以这里作为入口执行一连串的op。可以说整个PHP的功能特性都是通过这些op句柄完成的(当然这些句柄会间接调用其他模块中的功能)，那么这154个opcode如何对应到这些static int ZEND_FASTCALL  ZEND_*的执行句柄的呢？同样在这个文件中，可以看到zend_init_opcodes_handlers函数，这个函数初始化一个 static const opcode_handler_t labels[]数组，这个 labels数组就是handlers的一张表，这个表有近4000个项，有一个算法将一个opcode映射到这个表中的一个元素，算法同样在zend_vm_execute.h中可以找到，靠近文件结尾zend_vm_set_opcode_handler和zend_vm_get_opcode_handler就是这个算法的实现。

那么引擎是如何通过这些op handler实现PHP语言的特性的呢？这里我举一个最简单的例子，考虑下面只有一行的PHP代码：

    <?php
    	$a = 123;
    ?>

通过某种方法（以后再介绍这些方法）我们可以知道这行代码主要生成一个zend_op，其主要成员值为:

- opcode = 38  (对应#define ZEND_ASSIGN  38)
- op1       = $a ($a变量实际上是以cv形式存在，以后介绍)
- op2       = 123 (以const常量形式存在)

handler = ZEND_ASSIGN_SPEC_CV_CONST_HANDLER（得到这个handler的名字不是一件容易的事，以后给出方法）

opcode ZEND_ASSIGN的意思是将一个常量赋值给一个cv(compiled variable)，这个cv其实就是$a变量的一种存在形式。在zend_vm_execute.h中搜索到ZEND_ASSIGN_SPEC_CV_CONST_HANDLER的定义，其主要功能就是取op2的值123，将其赋值给op1的变量，当然这个过程比想象中的要复杂一些，会有变量的初始化，变量的写时赋值等过程，以后会介绍每一个过程。这样这条PHP语句的功能就完成了。可以看出，op handler只是按照一些固定的方式来对操作数op1 op2（可能还有result）进行操作，handler不理会这些操作数中的具体值，这些值是在编译阶段生成op的时候确定的，比如如果$a = 123 改成 $a =456，那么生成的op中op2就是456了，handler始终按照固定的方式来处理。

因此我们能知道，PHP的执行过程是先通过编译器将PHP代码编译成op code,然后然后zend虚拟机按照一定顺序执行这些opcode，具体是将每个opcode分发给特定的op code handler。

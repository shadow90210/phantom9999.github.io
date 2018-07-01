# 15-操作码OpCode
运行一段PHP代码主要有两个阶段：编译和执行。 当然编译过程中还包括词法分析语法分析不同阶段和细节，这里我们将其作为一个整体。在这两个阶段之间，PHP代码会被编译成op code，可以将其认为是引擎的一个中间语言，编辑阶段把PHP源码生成op code，然后在执行阶段执行这些op code。这篇文章将简单的介绍op code。

PHP代码编译之后会生成许多的op，每一个op都是一个zend_op类型的c变量。相关的定义可以在{PHPSRC}/Zend/zend_compile.h中看到：

    struct _zend_op {  
        opcode_handler_t handler;  
        znode result;  
        znode op1;  
        znode op2;  
        ulong extended_value;  
        uint lineno;  
        zend_uchar opcode;  
    };  
      
    typedef struct _zend_op zend_op;  

简单的说说这几个字段：

1.result,op1,op2

这三个字段都是znode类型，它们是op的操作数和操作结果载体，当然并不是每个op都需要使用这三个字段，根据op的功能不同，会使用其中某些字段。比如类型为ZEND_ECHO的op值需要使用op1，功能就是将op1中的相应的值输出。一会再单独介绍znode类型。

2.opcode

opcode的类型为zend_uchar，zend_uchar实际上就是unsigned char，此字段保存的整形值即为op的编号，用来区分不同的op类型，opcode的可取值都被定义成了宏，可以在{PHPSRC}/Zend/zend_vm_opcodes.h中看到这些宏的定义，类似如下：

    #define ZEND_NOP                               0  
    #define ZEND_ADD                               1  
    #define ZEND_SUB                               2  
    #define ZEND_MUL                               3  
    #define ZEND_DIV                               4  
    #define ZEND_MOD                               5  
    #define ZEND_SL                                6  
    #define ZEND_SR                                7  
    #define ZEND_CONCAT                            8  
    #define ZEND_BW_OR                             9  
    #define ZEND_BW_AND                           10  
    //......  

3.handler

op的执行句柄，其类型为opcode_handler_t，opcode_handler_t的类型定义为typedef int (ZEND_FASTCALL *opcode_handler_t) (ZEND_OPCODE_HANDLER_ARGS); 这个函数指针为op定义了执行方式，每一种opcode字段都对应一个种类的handler,比如opcode= 38 (ZEND_ASSIGN), 那么其对应的handler对应的就是static int ZEND_FASTCALL  ZEND_ASSIGN_**种类的handler，根据op操作数类型的不同，可以确定到这个种类中的某一个具体的函数，比如如果$a = 1;这样的代码生成的op，操作数为const和cv，最后就能确定handler为函数ZEND_ASSIGN_SPEC_CV_CONST_HANDLER，这些handler函数都定义在{PHPSRC}/Zend/zend_vm_execute.h中，此文件可以由一个PHP脚本生成，其中也定义了通过op来映射得到其hander的算法。

4.lineno

op对应源代码文件中的行号。

5.extended_value

扩展字段暂时不介绍
## 操作数znode简介 

操作数字段是这个类型中比较重要的部分了，其中op1,op2,result三个操作数定义为znode类型，znode相关定义在此文件中：

    typedef struct _znode {  
        int op_type;  
        union {  
            zval constant;  
      
            zend_uint var;  
            zend_uint opline_num; /*  Needs to be signed */  
            zend_op_array *op_array;  
            zend_op *jmp_addr;  
            struct {  
                zend_uint var;  /* dummy */  
                zend_uint type;  
            } EA;  
        } u;  
    } znode;  

znode类型中定义了两个字段：

1.op_type

这个int类型的字段定义znode操作数的类型，这些类型的可取值的宏定义在此文件中

    #define IS_CONST    (1<<0)  
    #define IS_TMP_VAR  (1<<1)  
    #define IS_VAR      (1<<2)  
    #define IS_UNUSED   (1<<3)    /* Unused variable */  
    #define IS_CV       (1<<4)    /* Compiled variable */  

- IS_CONST：表示常量，例如$a = 123; $b = "hello";这些代码生成OP后，123和"hello"都是以常量类型操作数存在。
- IS_TMP_VAR：表示临时变量，临时变量一般在前面加~来表示，这是一些OP执行过程中需要用到的中间变量，例如初始化一个数组的时候，就需要一个临时变量来暂时存储数组zval，然后将数组赋值给变量。
- IS_VAR： 一般意义上的变量，以$开发表示，此种变量本人目前研究的较少，暂不介绍
- IS_UNUSED ： 暂时不介绍，从名字来看应该是标识为不使用
- IS_CV：这种类型的操作数比较重要，此类型是在PHP后来的版本中(大概5.1)中才出现，CV的意思是compiled variable，即编译后的变量，变量都是保存在一个符号表中，这个符号表是一个哈希表，试想如果每次读写变量的时候都需要到哈希表中去检索，势必会对效率有一定的影响，因此在执行上下文环境中，会将一些编译期间生成的变量缓存起来，此过程以后再详细介绍。此类型操作数一般以!开头表示，比如变量$a=123;$b="hello"这段代码，$a和$b对应的操作数可能就是!0和!1, 0和1相当于一个索引号，通过索引号从缓存中取得相应的值。

2.u

此字段为一个联合体，根据op_type的不同，u取不同的值。比如op_type=IS_CONST的时候，u中的constant保存的就是操作数对应的zval结构。例如$a=123时，123这个操作数中，u中的constant是一个IS_LONG类型的zval,其值lval为123。

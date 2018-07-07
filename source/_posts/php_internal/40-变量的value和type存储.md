---
title: 40-变量的value和type存储
tags: php_internal
categories: php_internal
abbrlink: fab9b467
date: 2018-01-01 20:07:40
updated: 2018-01-01 20:07:40
---

# 40-变量的value和type存储
PHP是一种弱类型的脚本语言，弱类型不表示PHP的变量没有类型区分，PHP变量有8种原始类型：

四种标量类型：

- boolean（布尔型）
- integer（整型）
- float（浮点型）
- string（字符串）

两种复合类型：

- array（数组）
- object（对象）

两种特殊类型：

- resource（资源）
- NULL

一个变量能在运行期间从一种类型转换为另一种类型，那么PHP是如何实现这种变量的类型戏法的呢？

在引擎内部，变量都是用一个结构体来表示，这个结构体可以在{PHPSRC}/Zend/zend.h中找到：

    struct _zval_struct {  
        /* Variable information */  
        zvalue_value value;     /* value */  
        zend_uint refcount__gc;  
        zend_uchar type;    /* active type */  
        zend_uchar is_ref__gc;  
    };

这里我们暂时只关心 value和type两个成员，其中value是一个联合, 也就是变量的实际值，type是变量的动态类型，根据类型的不同，value使用不同的成员。

type的各种类型都被定义成了宏，同样在此文件中定义了这些宏：

    #define IS_NULL     0  
    #define IS_LONG     1  
    #define IS_DOUBLE   2  
    #define IS_BOOL     3  
    #define IS_ARRAY    4  
    #define IS_OBJECT   5  
    #define IS_STRING   6  
    #define IS_RESOURCE 7  
    #define IS_CONSTANT 8  
    #define IS_CONSTANT_ARRAY   9

    value的类型zvalue_value同样定义在此文件中：

    typedef union _zvalue_value {  
        long lval;                  /* long value */  
        double dval;                /* double value */  
        struct {  
            char *val;  
            int len;  
        } str;  
        HashTable *ht;              /* hash table value */  
        zend_object_value obj;  
    } zvalue_value;  

1.布尔类型变量。type=IS_BOOL，value中的lval字段为值，lval取值为(0,1)。在{PHPSRC}/Zend/zend_operators.h中定义了取一个zval的布尔值的宏：#define Z_BVAL(zval)   ((zend_bool)(zval).value.lval)

2.整形变量。type=IS_LONG, value中的lval字段为值。#define Z_LVAL(zval)   (zval).value.lval

3.浮点型变量。type=IS_DOUBLE, value中的dval字段为值。#define Z_DVAL(zval)   (zval).value.dval

4.字符串变量。type=IS_STRING,   value中的str结构有效，该结构中var为字符串的指针，len为字符串的长度。#define Z_STRVAL(zval)   (zval).value.str.val    //取字符串指针，#define Z_STRLEN(zval)   (zval).value.str.len    //取字符串长度

另外的复杂类型这里暂不详细介绍，对于数组类型，ht字段生效，ht是一个哈希表的指针，数组都是哈希表的形式存在，对于对象类型变量，obj保存这对象的信息，对于资源类型变量，用lval间接保存这资源得一个标识符。

这样引擎就用一个zval类型来实现了所有的PHP变量类型，知道以上原理之后，变量的类型转换就容易实现了，在{PHPSRC}/Zend/zend_operators.c中定义了各种类型转换的宏，比如转换成布尔类型的宏zendi_convert_to_boolean，主要的思路就是先把type变成IS_BOOL,然后根据原变量的不同类型的值按照一定规则转换成lval的0或则1。

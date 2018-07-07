---
title: 本文讲述14个经典的php面试题.
tags: php-devel
categories: php-devel
abbrlink: 329776f3
date: 2018-03-01 20:07:02
updated: 2018-03-01 20:07:02
---

本文讲述14个经典的php面试题.
## 问题一
### 问题描述
考虑下面代码:

    $str1 = 'yabadabadoo';
    $str2 = 'yaba';
    if (strpos($str1,$str2)) {
        echo "\"" . $str1 . "\" contains \"" . $str2 . "\"";
    } else {
        echo "\"" . $str1 . "\" does not contain \"" . $str2 . "\"";
    }

输出的结果是什么??为什么会出现这个结果?? 如何处理这种情况??

### 问题解答

输出结果为:

    "yabadabadoo" does not contain "yaba"

这个问题考察的是"strpos()"的使用.
这个函数返回变量\$str2在\$str1中的位置.
如果未找到就返回false.
但是, 本次调用时, 这个函数返回的值为0,
对于if语句来说, 0等同于false.

对于这种情况, 需要在if语句中判断函数返回值是否为false, 修改的代码如下:

    $str1 = 'yabadabadoo';
    $str2 = 'yaba';
    if (strpos($str1,$str2) !== false) {
        echo "\"" . $str1 . "\" contains \"" . $str2 . "\"";
    } else {
        echo "\"" . $str1 . "\" does not contain \"" . $str2 . "\"";
    }

注意, 在这里我们"!=="操作符, 而不是"!="操作符.
如果我们使用"!=", 我们就遇到这样一个问题, 0为强制转化为布尔表达式时, 0对应的是false.
所以"0 != false"的结果为"false".


## 问题二
### 问题描述
下面的代码的输出结果是什么??

    $x = 5;
    echo $x;
    echo "<br />";
    echo $x+++$x++;
    echo "<br />";
    echo $x;
    echo "<br />";
    echo $x---$x--;
    echo "<br />";
    echo $x;

### 问题解答
输出结果是:

    5
    11
    7
    1
    5

本题考察"后++"和"后--"的使用与"++"操作符和"+"操作符的优先级.

"后++"和"后--"先返回数值, 然后执行加一或者减一操作.
当同时出现"++"和"+"操作符时, 因为"++"操作符比"+"操作符高, 所以先执行"++"操作, 后执行"+"操作.


## 问题三
### 问题描述

执行完下面的代码后, 变量"\$a"和变量"\$b"的值时多少??

    $a = '1';
    $b = &$a;
    $b = "2$b";

### 问题解答

变量"\$a"和变量"\$b"的值都将等于字符串"21".
在执行代码"\$b = &\$a;"时, 变量"\$b"就赋值为变量"\$a"的引用, 变量"\$b"修改时, 变量"\$a"的值也将修改.
当变量赋值是出现双引号时, php将执行双引号中的表达式, 即"2\$b"的结果实际为字符串"2"和变量"\$b"的连接, 也就是字符串"21".

## 问题四
### 问题描述
下面各行的输出结果是什么???

    var_dump(0123 == 123);
    var_dump('0123' == 123);
    var_dump('0123' === 123);

### 问题解答
运行的结果如下:

    bool(false)
    bool(true)
    bool(false)

对于语句"var_dump(0123 == 123)", 输出结果为"bool(false)".
因为以0开头的数组在php解释器看来, 是八进制数字.
八进制的"123"与十进制的"123"不相等.

对于语句"var_dump('0123' == 123)", 其输出结果为"bool(true)".
因为字符串"0123"与数值"123"比较时, 字符串"0123"将自动转化为数值型, 字符串中的"0"将被忽略.
所以这个字符串将被转化为数值类型的"123". 这个数值与数值"123"相等, 最后返回的结果为true.

对于语句"var_dump('0123' === 123)", 其输出结果为"bool(false)".
在这个语句中引入了严格比较符"===", 这个比较符会先比较两个变量的数据类型.
对于当前语句而言, 前者为字符串, 后者为数值型, 两个变量的类型不同, 所以返回false.


## 问题五

### 问题描述

下面的代码的结果是什么???有什么问题???如何修改??

    $referenceTable = array();
    $referenceTable['val1'] = array(1, 2);
    $referenceTable['val2'] = 3;
    $referenceTable['val3'] = array(4, 5);

    $testArray = array();

    $testArray = array_merge($testArray, $referenceTable['val1']);
    var_dump($testArray);
    $testArray = array_merge($testArray, $referenceTable['val2']);
    var_dump($testArray);
    $testArray = array_merge($testArray, $referenceTable['val3']);
    var_dump($testArray);

### 问题解答
输出结果如下所是:

    array(2) { [0]=> int(1) [1]=> int(2) }
    NULL
    NULL

同时将会出现两个警告:

    Warning: array_merge(): Argument #2 is not an array
    Warning: array_merge(): Argument #1 is not an array

本题的关键是, 如果"array_merge()"函数的两个参数中有一个不是数组类型, 这个函数的返回值将返回"NULL".
在一般逻辑下, 形如语句"array_merge(\$someValidArray, NULL)"的结果为"\$someValidArray",
然而, 这个函数实际的返回结果是"NULL", 而且, 这一点在php的官方文档中也很少提到.



As a result, the call to $testArray = array_merge($testArray, $referenceTable['val2']) evaluates
to $testArray = array_merge($testArray, 3) and,
since 3 is not of type array, this call to array_merge() returns NULL,
which in turn ends up setting $testArray equal to NULL.
Then, when we get to the next call to array_merge(), $testArray is now NULL
so array_merge() again returns NULL.
(This also explains why the first warning complains about argument #2
and the second warning complains about argument #1.)

执行语句"\$testArray = array_merge(\$testArray, \$referenceTable['val2'])"时,
这个语句将转化为"\$testArray = array_merge(\$testArray, 3)".
由于上面提到的原因, 变量"\$testArray"的值被赋值为"NULL".
执行语句"\$testArray = array_merge(\$testArray, \$referenceTable['val3']);"时,
这个语句等同于"\$testArray = array_merge(NULL, \$referenceTable['val3']);".
这个语句中, 第一个参数的值为NULL, 不是数组类型, 函数调用的结果为"NULL".
所以, 最后变量"\$testArray"的值为"NULL".
这也说明了, 为什么两个警告中, 一个时参数2, 一个是参数1了.


面对这种情况, 修改的措施也是简单的.
直接对参数进行类型转化, 我们就可以实现原有的要求:

    $testArray = array_merge($testArray, (array)$referenceTable['val1']);
    var_dump($testArray);
    $testArray = array_merge($testArray, (array)$referenceTable['val2']);
    var_dump($testArray);
    $testArray = array_merge($testArray, (array)$referenceTable['val3']);
    var_dump($testArray);

这样的输出结果如下:

    array(2) { [0]=> int(1) [1]=> int(2) }
    array(3) { [0]=> int(1) [1]=> int(2) [2]=> int(3) }
    array(5) { [0]=> int(1) [1]=> int(2) [2]=> int(3) [3]=> int(4) [4]=> int(5) }





## 问题六
### 问题描述
下面代码输出的结果是什么??为什么??

    $x = true and false;
    var_dump($x);

### 问题解答

出乎意料的, 上面的代码将输出"bool(true)", 看似"and"操作符表现的跟"or"操作符相同 .

The issue here is that the = operator takes precedence over the and operator in order of operations, so the statement $x = true and false ends up being functionally equivalent to:

事实上, "="操作符的优先级比"and"操作符高.
因此, 语句"$x = true and false"可以等同下面语句:

    $x = true;       // 将"$x"变量赋值为true, 由于赋值成功, 返回true.
    true and false;  // 结果是false, 但是这个结果与$x无关.

为了使自己的表达式更加清晰, 适当添加括号是一个很好的习惯.
例如, 形如上面的表达式"\$x = true and false"用"\$x = (true and false)"代替,
那么"\$x"就被设置为"false"了.


## 问题七
### 问题描述
下面运行下面的语句, 变量"\$x"的值为:

    $x = 3 + "15%" + "$25"

### 问题解决
变量"\$x"的值为18.

Here’s why:

PHP supports automatic type conversion based on the context in which a variable or value is being used.

php支持根据语境自动变量类型转化.

如果你在一个表达式中使用算术运算, 并且这个表达式中包含字符串.
那么这些字符串将被转化为相应的数字, 以完成算术运算.
如果一个字符串由数字开头, 那么这个字符串将忽略非数字部分, 将数字部分的字符转化为相应的数值.
如果一个字符串以非数字开头, 那么这个字符串将被转化为0.

基于上面的原因, 字符串"15%"将被转化为"15", 字符串"$25"将被转化为"0".
那么原来的计算表达式"\$x = 3 + "15%" + "\$25""将转化为"\$x=3+15+0", 最后的结果为"18".


## 问题八

执行下面代码后, 变量"\$text"的值是什么, 语句"strlen(\$text)"的返回值是什么??

    $text = 'John ';
    $text[10] = 'Doe';

执行完上面的代码, 变量"\$text"的值为"John      D", 即字符串"john"后面跟着五个空格, 然后加一个字符"D".
并且代码"strlen(\$text)"的返回值为11.

这里有两点需要注意:

首先, 变量"\$text"是一个字符串, 设置这个变量中的一个元素的值, 就是设置这个变量中的一个字符.
代码"$text[10] = 'Doe'", 用于替换的字符为原字符中的第一个, 即"Doe"中的'D'.

第二, 代码"\$text[10] = 'Doe'"运行时, 将修改变量'\$text'的第11个字符, 将其设置为'D'.
更重要的是, 原始字符串的长度为5, 对于其他编译器或者解释器而言, 这种操作会出现数组越界情况.
但是对于php而言, 这个操作是允许的.
php会自动适应这种情况, 并将其他字符位置置空.


## 问题九
### 问题描述
"PHP_INT_MAX"是php的一个常量, 用于标识所支持的最大整数(这个值与php的版本和操作系统的架构有关).

假设代码"var_dump(PHP_INT_MAX)"的输出值为"9223372036854775807".

依据上面这种情况, 下面的代码的运行结果是什么??

    var_dump(PHP_INT_MAX + 1)
    var_dump((int)(PHP_INT_MAX + 1))

### 问题解答

代码"var_dump(PHP_INT_MAX + 1)"的结果将被转化为double类型, 在上面的情况中, 输出的结果为"double(9.2233720368548E+18)".
这个问题的关键是考察面试者对于php对于大整数的处理, 即将整型无法处理的数据转化为double类型.

代码"var_dump((int)(PHP_INT_MAX + 1))"将输出结果为一个负数, 在上面那种情况下, 输出的结果为"int(-9223372036854775808))".


## 问题十
### 问题描述

如何对一个字符串数组根据值在忽略大小写的情况下进行排序, 并不打乱键与值的关系.
例如给出下面的数组:

    array(
        '0' => 'z1',
        '1' => 'Z10',
        '2' => 'z12',
        '3' => 'Z2',
        '4' => 'z3',
    )

排序完的结果为:

    array(
        '0' => 'z1',
        '3' => 'Z2',
        '4' => 'z3',
        '1' => 'Z10',
        '2' => 'z12',
    )

The trick to solving this problem is to use three special flags with the standard asort() library function:

解决这个问题的关键是, 在"asort()"库函数中设置特殊的标识:

    asort($arr, SORT_STRING|SORT_FLAG_CASE|SORT_NATURAL)

函数"asort()"函数是标准函数"sort()"的一个变形, 这个函数会保持原有数据的键值关系.
三个标识"SORT_STRING", "SORT_FLAG_CASE"和"SORT_NATURAL"设置了排序的要求,
将元素视为字符串, 忽略大小写, 保持原有的键值关系.

注意, 使用函数"natcasesort()"并不能得到预期的效果, 因为它会打乱原有的键值关系.


## 问题十一
### 问题描述

在php中, PEAR是什么???

### 问题解答

PEAR(PHP Extension and Application Repository)是一个可复用的php组件库.
这个资源库中包含了各种各样的php代码摘要和库.

PEAR同时提供了一个自动下载包的命令行工具.





## 问题十二

What are the differences between echo and print in PHP?

### 问题描述
在php中"echo"和"print"的区别是什么???


echo and print are largely the same in PHP. Both are used to output data to the screen.

### 问题解答
在php中, "echo"和"print"有着及其相似的功能.
他们都能将数据展现在屏幕上.

他们的不同如下所是:


The only differences are as follows:

- "echo"使用时, 没有返回值, 而"print"会返回1. 这也使得"print"可以在表达式中使用.
- "echo"可以接受多个参数, 但是, 这种方式使用很少. "print"只接收一个参数
- "echo"是一个关键字, 而"print"是一个特殊的函数.




## 问题十三
### 问题描述

下面的代码段的结果是什么:

    $v = 1;
    $m = 2;
    $l = 3;

    if( $l > $m > $v){
        echo "yes";
    }else{
        echo "no";
    }

### 问题解答
一般人会认为$3 > 2 > 1$, 那么最后输出的结果为"yes".
事实上, 最后输出的结果为"no".

首先, 式子"\$l > \$m"将被执行, 并返回一个布尔值1, 即true.
接着比较这个布尔值和一个整型1, 即式子等同于"bool(1) > \$v", 结果为NULL, 所以最后的结果是"no".



## 问题十四
### 问题描述
当执行完下面的代码后, 变量"\$x"的值是什么?

    $x = NULL;

    if ('0xFF' == 255) {
        $x = (int)'0xFF';
    }
### 问题解答
最后的答案既不是NULL, 也不是255.
最后的结果是"\$x"为0.

首先, 我们需要判断表达式"'0xFF' == 255"的结果是true还是'false'.
当一个十六进制的字符串与一个整型数据比较时, 这个字符串将转为一个整型.
在内部, php会使用"is_numeric_string"判断这个字符串, 并将它转化为一个整型(因为另一个时参数时整型).
在这种情况下, 字符串"0xFF"将转化为255.
这时, 255与255比较, 结果为true.
这种情况只支持十六进制的字符串, 不支持八进制和二进制的字符串.

变量'\$x'的值并不是'0xFF'的整型值.
在显式的将字符串转化为整型时, 底层将使用"convert_to_long"函数, 而不是"is_numeric_string"函数.
函数"convert_to_long"对字符串进行转化时, 从左往右依次转化, 直到遇到第一个非数值字符.
对于'0xFF'字符, 第一个非数字字符是'x'.
那么整个转化过程的结果为'0'.
所以"(int)'0xFF'"的结果是0, 即'\$x'的值为0.

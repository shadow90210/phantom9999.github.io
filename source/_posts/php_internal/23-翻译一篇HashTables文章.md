---
title: 23-翻译一篇HashTables文章
tags: php_internal
categories: php_internal
date: 2018-01-01 20:07:23
updated: 2018-01-01 20:07:23
---

# 23-翻译一篇HashTables文章
In case you ever heard me talking about PHP internals I certainly mentioned something along the lines of "Everything in PHP is a HashTable" that's not true, but next to a zval the HashTable is one of the most important structures in PHP. While preparing my "PHP Under The Hood" talk for the Dutch PHP Conference there was a question on IRC about extension_loaded() being faster thanfunction_exists(), which might be strange as both of them are simple hash lookups and a hash lookup is said to be O(1). I started to write some slides for it but figured out that I won't have the time to go through it during that presentation, so I'm doing this now, here:

如果你以前看过我写的PHP内核相关的内容，那么你一定记得我说过PHP中的任何东西都是哈希表。这不一定是真的，但是除了zval，哈希表确实是PHP中的最重要的结构了。在为Dutch PHP 大会准备"PHP Under The Hood" 演讲时，IRC上有个关于extension_loaded()比function_exists()快的问题，这让人感到疑惑，它们底层都是简单的哈希查找，而哈希查找的时间复杂度为O(1)。我写了一些相关的幻灯片，但是那个大会上已经没有时间了，所以放在了这里。

You all should know PHP arrays. They allow you to create a list of elements where every element may be identified by a key. This key may either be an integer or a string. Now we need a way to store this association in an effective way in memory. An efficient way to store a collection of data in memory is a "real" array, an array of elements indexed by a sequence of numbers, starting at 0, up to n. As memory is essentially a sequence of numbered storage blocks (this obviously is simplified, there might be segments and offsets, there might be virtual pages, etc.) we can efficiently access an element by knowing the address of the first element, the size of the elements (we assume all have the same size, which is the case here), and the index: The address of the n-th element is start_address + (n * size_of_element). That's basically what C arrays are.

大家都应该知道PHP数组，它可以保存一系列元素，并且可以用一个key来标识每个元素，这个key可以是整数或字符串。因而我们需要在内存中有一个高效的方案来保存这种关联关系。其中一个高效的方案是把内存中的这些数据存储为”真正“的数组，一个从索引0开始到n的连续序列。内存从本质上讲就是一个连续的存储块（这么说是简化了的结果，其实这里有段、偏移以及虚拟页等概念）。我们可以快速地访问一个元素：只要我们知道第一个元素的地址，元素的大小（这里假设每个元素大小一致）和待查找元素的索引值。第n个元素的地址计算公式为：第一个元素地址 + (n * 元素大小)，C语言中的数组就是这么实现的。

Now we're not dealing with C and C arrays but also want to use string offsets so we need a function to calculate a numeric value, a hash, for each key. An hash function you most likely know is MD5, MD5 is creating a 128 bit numeric value which is often represented using 32 hexadecimal characters. For our purpose 128 bit is a bit much and MD5 is too slow so the PHP developers have chosen the "DJBX33A (Daniel J. Bernstein, Times 33 with Addition)" algorithm. This hash function is relatively fast and gives us an integer of the value, the trouble with this algorithm is that it is more likely to produce conflicts, that means to string values which create the same numeric value.

现在我们处理的不是C语言和它的数组，我们想用字符串偏移量，所以我们需要一个哈希函数，为每个键值计算出一个数值。大家最熟悉的哈希函数可能是MD5，它产生一个128比特的数值，且通常用32个十六进制的字符表示。对于我们的目的而言，128比特的数值太大了，MD5计算过程缓慢，因而PHP开发者选择了"DJBX33A (Daniel J. Bernstein, Times 33 with Addition)"算法。这个哈希函数相对快地从一个字符串计算出一个整数，缺点是冲突的概率增大了，即不同的字符串产生了相同的数值。

Now back to our C array: For being able to safely read any element, to see whether it is used, we need to pre-allocate the entire array with space for all possible elements. Given our hash function returning a system dependent (32 bit or 64 bit) integer this is quite a lot (size of an element multiplied wit the max int value), so PHP does another trick: It throws some digits away. For this a table mask is calculated. The a table mask is a number which is a power of two and then one subtracted and ideally higher than the number elements in the hash table. If one looks at this in binary representation this is a number where all bits are set. Doing a binary AND operation of our hash value and the table this gives us a number which is smaller than our table mask. Let's look at an example: The hash value of "foobar" equals, in decimal digits, to 3127933054. We assume a table mask of 2047 (2¹¹-1).

回到C数组：为了安全地读到任何一个元素，不管它是否用到，我们需要为所有可能元素预先分配空间。假设我们的哈希函数返回一个系统相关（32位或64位）的整数，数值非常大，这时PHP需要另外一个技巧：抛弃一些比特。这是通过表掩码实现的，表掩码是这样一个数，不小于哈希表的元素个数值且为2的次方减一。如果查看它的二进制表现形式，即各个位都设置为1的数。把哈希值和表掩码做二进制与操作，得到一个小于表掩码的数值。如："foobar"的十进制格式的哈希值为3127933054，假设表掩码为2047 (2¹¹-1)。

    3127933054    10111010011100000111100001111110
    & 2047 00000000000000000000011111111111
    =        126    00000000000000000000000001111110

Wonderful - we have an array Index, 126, for this string and can set the value in memory!

好——我们得到这个字符串的数组索引值126，就可以在内存中操作了。

If life were that easy. Remember: We used a hashing function which is by far not collision free and then dropped almost two thirds of the binary digits by using the table mask. This makes it rather likely that some collisions appear. These collisions are handled by storing values with the same hash in a linked list. So for accessing the an element one has to

1. Calculate the hash
2. Apply the table mask
3. locate the memory offset
4. check whether this is the element we need, traverse the linked list.

生活并不如此简单，我们使用的哈希函数不能避免碰撞，并通过表掩码抛弃了几乎三分之二的位，这更增大了碰撞的几率。解决办法是把相同哈希值的元素保存在链表中，所以访问一个元素的时候：

1. 计算哈希值
2. 应用表掩码
3. 找到内存位置
4. 遍历链表，检查是否是查找的元素

Now the question initially asked was why extension_loaded() might be faster than function_exists() and we can tell: For many random reasons and since you have chosen a value which probably conflicts in the first, not in the second. So now the question is how often such collisions happen for this I did a quick analysis: Given the function table of a random PHP build of mine with 1106 functions listed I have 634 unique hash values and 210 hash values calculated from different functions. Worst is the value of 471 which represents 6 functions.

现在可以回答文章开头提出的为什么extension_loaded()可能比function_exists() 快的问题了：它可以在第一次查找时即找到对应元素。现在的问题是这种冲突的概率有多大，我做了一个快速的分析：我的PHP构建版本的函数表中有1106个函数，计算得到634个唯一的哈希值，210个有冲突的哈希值，最差的哈希值471对应了6个函数。

Full results are online but please mind: These results are very specific to my environment. Also mind that the code actually works on a copy of my function table so the table mask might be different, which changes results. Also note that the given PHP code won't work for you as I added special functions exporting the table mask and hash function to mz version of PHP. And then yet another note: doing performance optimizations on this level is the by far worst place as to many unknown factors go in. And you don't have any measurable performance win. Mind readability. Clear code is worth way more than the 2 CPU cycles you probably gain! But you may have learned how hash tables work.

全部的结果在这里，不过请注意，这个结果是在我的环境里测试的，而且即使是我这个版本的代码拷贝，也会因为表掩码不同而导致结果不同。另外要注意的是，这些代码在你的环境中是无法工作的，因为我在PHP源码中添加了一些表掩码和哈希函数相关的代码。另外一个提示：由于各种未知的因素，想在这个地方进行优化并不合适，你很难获得明显的性能提升。清晰可读的代码远比两个CPU周期重要，不过通过它你可以学习哈希表是如何工作的。

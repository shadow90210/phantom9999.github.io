# 22-PHP哈希算法设计
Hash Table是PHP的核心，这话一点都不过分。PHP的数组、关联数组、对象属性、函数表、符号表等等都是用HashTable来做为容器的。

PHP的HashTable采用的拉链法来解决冲突，这个自不用多说，我今天主要关注的就是PHP的Hash算法，和这个算法本身透露出来的一些思想。

PHP的Hash采用的是目前最为普遍的DJBX33A (Daniel J. Bernstein, Times 33 with Addition)，这个算法被广泛运用与多个软件项目，Apache、Perl和Berkeley DB等。对于字符串而言这是目前所知道的最好的哈希算法，原因在于该算法的速度非常快，而且分类非常好(冲突小，分布均匀)。

算法的核心思想就是：

    hash(i) = hash(i-1) * 33 + str[i]

在zend_hash.h中，我们可以找到在PHP中的这个算法：

    static inline ulong zend_inline_hash_func(char *arKey, uint nKeyLength)
    {
        register ulong hash = 5381;
     
        /* variant with the hash unrolled eight times */
    	for (; nKeyLength >= 8; nKeyLength -= 8) {
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
            hash = ((hash << 5) + hash) + *arKey++;
        }
        switch (nKeyLength) {
            case 7: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
            case 6: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
            case 5: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
            case 4: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
            case 3: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
            case 2: hash = ((hash << 5) + hash) + *arKey++; /* fallthrough... */
            case 1: hash = ((hash << 5) + hash) + *arKey++; break;
            case 0: break;
    EMPTY_SWITCH_DEFAULT_CASE()
        }
        return hash;
    }

    相比在Apache和Perl中直接采用的经典Times 33算法：

      hashing function used in Perl 5.005:
      # Return the hashed value of a string: $hash = perlhash("key")
      # (Defined by the PERL_HASH macro in hv.h)
      sub perlhash
      {
          $hash = 0;
          foreach (split //, shift) {
              $hash = $hash*33 + ord($_);
          }
          return $hash;
      }

在PHP的hash算法中，我们可以看出很处细致的不同。首先，最不一样的就是，PHP中并没有使用直接乘33，而是采用了：

    hash << 5 + hash

这样当然会比用乘快了。

然后，特别要主意的就是使用的unrolled，我前几天看过一篇文章讲Discuz的缓存机制，其中就有一条说是Discuz会根据帖子的热度不同采用不同的缓存策略，根据用户习惯，而只缓存帖子的第一页(因为很少有人会翻帖子)。

于此类似的思想，PHP鼓励8位一下的字符索引，他以8为单位使用unrolled来提高效率，这不得不说也是个很细节的，很细致的地方。

另外还有inline，register变量 … 可以看出PHP的开发者在hash的优化上也是煞费苦心。

最后就是，hash的初始值设置成了5381，相比在Apache中的times算法和Perl中的Hash算法(都采用初始hash为0)，为什么选5381呢？具体的原因我也不知道，但是我发现了5381的一些特性：

    Magic Constant 5381:
      1. odd number
      2. prime number
      3. deficient number
      4. 001/010/100/000/101 b

看了这些，我有理由相信这个初始值的选定能提供更好的分类。

至于说，为什么是Times 33而不是Times 其他数字，在PHP Hash算法的注释中也有一些说明，希望对有兴趣的同学有用：

    DJBX33A (Daniel J. Bernstein, Times 33 with Addition)

    This is Daniel J. Bernstein's popular `times 33' hash function as
    posted by him years ago on comp.lang.c. It basically uses a function
    like ``hash(i) = hash(i-1) * 33 + str[i]''. This is one of the best
    known hash functions for strings. Because it is both computed very
    fast and distributes very well.

    The magic of number 33, i.e. why it works better than many other
    constants, prime or not, has never been adequately explained by
    anyone. So I try an explanation: if one experimentally tests all
    multipliers between 1 and 256 (as RSE did now) one detects that even
    numbers are not useable at all. The remaining 128 odd numbers
    (except for the number 1) work more or less all equally well. They
    all distribute in an acceptable way and this way fill a hash table
    with an average percent of approx. 86%.

    If one compares the Chi^2 values of the variants, the number 33 not
    even has the best value. But the number 33 and a few other equally
    good numbers like 17, 31, 63, 127 and 129 have nevertheless a great
    advantage to the remaining numbers in the large set of possible
    multipliers: their multiply operation can be replaced by a faster
    operation based on just one shift plus either a single addition
    or subtraction operation. And because a hash function has to both
    distribute good _and_ has to be very fast to compute, those few
    numbers should be preferred and seems to be the reason why Daniel J.
    Bernstein also preferred it.

                   -- Ralf S. Engelschall <rse@engelschall.com>

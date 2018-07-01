# 08-Apache模块介绍
## Apache概述

Apache是目前世界上使用最为广泛的一种Web Server，它以跨平台、高效和稳定而闻名。按照去年官方统计的数据，Apache服务器的装机量占该市场60%以上的份额。尤其是在X（Unix/Linux）平台上，Apache是最常见的选择。其它的Web Server产品，比如IIS，只能运行在Windows平台上，是基于微软.Net架构技术的不二选择。

Apache支持许多特性，大部分通过模块扩展实现。常见的模块包括mod_auth（权限验证）、mod_ssl（SSL和TLS支持） mod_rewrite（URL重写）等。一些通用的语言也支持以Apache模块的方式与Apache集成。 如Perl，Python，Tcl，和PHP等。

Apache并不是没有缺点，它最为诟病的一点就是变得越来越重，被普遍认为是重量级的WebServer。所以，近年来又涌现出了很多轻量级的替代产品，比如lighttpd，nginx等等，这些WebServer的优点是运行效率很高，但缺点也很明显，成熟度往往要低于Apache，通常只能用于某些特定场合。
## Apache组件逻辑图

Apache是基于模块化设计的，总体上看起来代码的可读性高于php的代码，它的核心代码并不多，大多数的功能都被分散到各个模块中，各个模块在系统启动的时候按需载入。你如果想要阅读Apache的源代码，建议你直接从main.c文件读起，系统最主要的处理逻辑都包含在里面。

MPM（Multi -Processing Modules，多重处理模块）是Apache的核心组件之一，Apache通过MPM来使用操作系统的资源，对进程和线程池进行管理。Apache为了能够获得最好的运行性能，针对不同的平台(Unix/Linux、Window)做了优化，为不同的平台提供了不同的MPM，用户可以根据实际情况进行选择，其中最常使用的MPM有prefork和worker两种。至于您的服务器正以哪种方式运行，取决于安装Apache过程中指定的MPM编译参数，在X系统上默认的编译参数为prefork。由于大多数的Unix都不支持真正的线程，所以采用了预派生子进程(prefork)方式，象Windows或者Solaris这些支持线程的平台，基于多进程多线程混合的worker模式是一种不错的选择。对此感兴趣的同学可以阅读有关资料，此处不再多讲。Apache中还有一个重要的组件就是APR（Apache portable Runtime Library），即Apache可移植运行库，它是一个对操作系统调用的抽象库，用来实现Apache内部组件对操作系统的使用，提高系统的可移植性。Apache对于php的解析，就是通过众多Module中的php Module来完成的。

<center>
![](images/2012_02_02_11.jpg)
</center>
<center>
Apache的逻辑构成以及与操作系统的关系
</center>
## PHP与Apache

当PHP需要在Apache服务器下运行时，一般来说，它可以mod_php5模块的形式集成， 此时mod_php5模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求， 然后将处理后的结果返回给Apache。如果我们在Apache启动前在其配置文件中配置好了PHP模块（mod_php5）， PHP模块通过注册apache2的ap_hook_post_config挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。

除了这种启动时的加载方式，Apache的模块可以在运行的时候动态装载， 这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。 我们所需要做的仅仅是给服务器发送信号HUP或者AP_SIG_GRACEFUL通知服务器重新载入模块。 但是在动态加载之前，我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。 Apache中对动态链接库的处理是通过模块mod_so来完成的，因此mod_so模块不能被动态加载， 它只能被静态编译进Apache的核心。这意味着它是随着Apache一起启动的。

Apache是如何加载模块的呢？我们以前面提到的mod_php5模块为例。 首先我们需要在Apache的配置文件httpd.conf中添加一行：

    LoadModule php5_module modules/mod_php5.so

这里我们使用了LoadModule命令，该命令的第一个参数是模块的名称，名称可以在模块实现的源码中找到。 第二个选项是该模块所处的路径。如果需要在服务器运行时加载模块， 可以通过发送信号HUP或者AP_SIG_GRACEFUL给服务器，一旦接受到该信号，Apache将重新装载模块， 而不需要重新启动服务器。

在配置文件中添加了所上所示的指令后，Apache在加载模块时会根据模块名查找模块并加载， 对于每一个模块，Apache必须保证其文件名是以“mod_”开始的，如PHP的mod_php5.c。 如果命名格式不对，Apache将认为此模块不合法。Apache的每一个模块都是以module结构体的形式存在， module结构的name属性在最后是通过宏STANDARD20_MODULE_STUFF以__FILE__体现。 关于这点可以在后面介绍mod_php5模块时有看到。这也就决定了我们的文件名和模块名是相同的。 通过之前指令中指定的路径找到相关的动态链接库文件后，Apache通过内部的函数获取动态链接库中的内容， 并将模块的内容加载到内存中的指定变量中。

在真正激活模块之前，Apache会检查所加载的模块是否为真正的Apache模块， 这个检测是通过检查module结构体中的magic字段实现的。 而magic字段是通过宏STANDARD20_MODULE_STUFF体现，在这个宏中magic的值为MODULE_MAGIC_COOKIE， MODULE_MAGIC_COOKIE定义如下：

    #define MODULE_MAGIC_COOKIE 0x41503232UL /* "AP22" */

最后Apache会调用相关函数(ap_add_loaded_module)将模块激活， 此处的激活就是将模块放入相应的链表中(ap_top_modules链表： ap_top_modules链表用来保存Apache中所有的被激活的模块，包括默认的激活模块和激活的第三方模块。）

# 10-Apache运行与钩子函数
Apache是目前世界上使用最为广泛的一种Web Server，它以跨平台、高效和稳定而闻名。按照去年官方统计的数据，Apache服务器的装机量占该市场60%以上的份额。尤其是在X（Unix/Linux）平台上，Apache是最常见的选择。其它的Web Server产品，比如IIS，只能运行在Windows平台上，是基于微软.Net架构技术的不二选择。

Apache并不是没有缺点，它最为诟病的一点就是变得越来越重，被普遍认为是重量级的WebServer。所以，近年来又涌现出了很多轻量级的替代产品，比如lighttpd,nginx等等，这些WebServer的优点是运行效率很高，但缺点也很明显，成熟度往往要低于Apache，通常只能用于某些特定场合。
## Apache的运行过程

Apache的运行分为启动阶段和运行阶段。 在启动阶段，Apache为了获得系统资源最大的使用权限，将以特权用户root（*nix系统）或超级管理员Administrator(Windows系统)完成启动， 并且整个过程处于一个单进程单线程的环境中。 这个阶段包括配置文件解析(如http.conf文件)、模块加载(如mod_php，mod_perl)和系统资源初始化（例如日志文件、共享内存段、数据库连接等）等工作。

Apache的启动阶段执行了大量的初始化操作，并且将许多比较慢或者花费比较高的操作都集中在这个阶段完成，以减少了后面处理请求服务的压力。

在运行阶段，Apache主要工作是处理用户的服务请求。 在这个阶段，Apache放弃特权用户级别，使用普通权限，这主要是基于安全性的考虑，防止由于代码的缺陷引起的安全漏洞。 Apache对HTTP的请求可以分为连接、处理和断开连接三个大的阶段。同时也可以分为11个小的阶段，依次为： Post-Read-Request，URI Translation，Header Parsing，Access Control，Authentication，Authorization， MIME Type Checking，FixUp，Response，Logging，CleanUp
## Apache Hook机制

Apache的Hook机制是指：Apache 允许模块(包括内部模块和外部模块，例如mod_php5.so,mod_perl.so等)将自定义的函数注入到请求处理循环中。换句话说，模块可以在Apache的任何一个处理阶段中挂接(Hook)上自己的处理函数，从而参与Apache的请求处理过程。

mod_php5.so/ php5apache2.dll就是将所包含的自定义函数，通过Hook机制注入到Apache中，在Apache处理流程的各个阶段负责处理php请求。

关于Hook机制在Windows系统开发也经常遇到，在Windows开发既有系统级的钩子，又有应用级的钩子。常见的翻译软件（例如金山词霸等等）的屏幕取词功能，大多数是通过安装系统级钩子函数完成的，将自定义函数替换gdi32.dll中的屏幕输出的绘制函数。

Apache 服务器的体系结构的最大特点，就是高度模块化。如果你为了追求处理效率，可以把这些dso模块在apache编译的时候静态链入，这样会提高Apache 5%左右的处理性能。
## Apache请求处理循环

Apache请求处理循环的11个阶段都做了哪些事情呢？

1. Post-Read-Request阶段。在正常请求处理流程中，这是模块可以插入钩子的第一个阶段。对于那些想很早进入处理请求的模块来说，这个阶段可以被利用。
2. URI Translation阶段。Apache在本阶段的主要工作：将请求的URL映射到本地文件系统。模块可以在这阶段插入钩子，执行自己的映射逻辑。mod_alias就是利用这个阶段工作的。
3. Header Parsing阶段。Apache在本阶段的主要工作：检查请求的头部。由于模块可以在请求处理流程的任何一个点上执行检查请求头部的任务，因此这个钩子很少被使用。mod_setenvif就是利用这个阶段工作的。
4. Access Control阶段。 Apache在本阶段的主要工作：根据配置文件检查是否允许访问请求的资源。Apache的标准逻辑实现了允许和拒绝指令。mod_authz_host就是利用这个阶段工作的。
5. Authentication阶段。Apache在本阶段的主要工作：按照配置文件设定的策略对用户进行认证，并设定用户名区域。模块可以在这阶段插入钩子，实现一个认证方法。
6. Authorization阶段。 Apache在本阶段的主要工作：根据配置文件检查是否允许认证过的用户执行请求的操作。模块可以在这阶段插入钩子，实现一个用户权限管理的方法。
7. MIME Type Checking阶段。Apache在本阶段的主要工作：根据请求资源的MIME类型的相关规则，判定将要使用的内容处理函数。标准模块mod_negotiation和mod_mime实现了这个钩子。
8. FixUp阶段。这是一个通用的阶段，允许模块在内容生成器之前，运行任何必要的处理流程。和Post_Read_Request类似，这是一个能够捕获任何信息的钩子，也是最常使用的钩子。
9. Response阶段。Apache在本阶段的主要工作：生成返回客户端的内容，负责给客户端发送一个恰当的回复。这个阶段是整个处理流程的核心部分。
10. Logging阶段。Apache在本阶段的主要工作：在回复已经发送给客户端之后记录事务。模块可能修改或者替换Apache的标准日志记录。 
11. CleanUp阶段。 Apache在本阶段的主要工作：清理本次请求事务处理完成之后遗留的环境，比如文件、目录的处理或者Socket的关闭等等，这是Apache一次请求处理的最后一个阶段。

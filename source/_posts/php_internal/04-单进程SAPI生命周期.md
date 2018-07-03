---
title: 04-单进程SAPI生命周期
tags: php_internal
categories: php
---

# 04-单进程SAPI生命周期
CLI/CGI模式的PHP属于单进程的SAPI模式。这类的请求在处理一次请求后就关闭。也就是只会经过如下几个环节： 开始 - 请求开始 - 请求关闭 - 结束 SAPI接口实现就完成了其生命周期。

<center>
![](/images/2012_02_02_05.jpg)
</center>

单进程多请求则如下图所示：

<center>
![](/images/2012_02_02_06.jpg)
</center>

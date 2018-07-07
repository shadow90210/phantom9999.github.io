---
title: 05-多进程⁄线程的SAPI生命周期
tags: php_internal
categories: php_internal
abbrlink: d22489dc
date: 2018-01-01 20:07:05
updated: 2018-01-01 20:07:05
---

# 05-多进程⁄线程的SAPI生命周期
## 多进程的SAPI生命周期

通常PHP是编译为apache的一个模块来处理PHP请求。Apache一般会采用多进程模式， Apache启动后会fork出多个子进程，每个进程的内存空间独立，每个子进程都会经过开始和结束环节， 不过每个进程的开始阶段只在进程fork出来以来后进行，在整个进程的生命周期内可能会处理多个请求。 只有在Apache关闭或者进程被结束之后才会进行关闭阶段，在这两个阶段之间会随着每个请求重复请求开始-请求关闭的环节。

<center>
![](/images/2012_02_02_07.jpg)
</center>
<center>
多进程SAPI生命周期
</center>

## 多线程的SAPI生命周期

多线程模式和多进程中的某个进程类似，不同的是在整个进程的生命周期内会并行的重复着 请求开始-请求关闭的环节。

<center>
![](/images/2012_02_02_08.jpg)
</center>

<center>
多线程SAPI生命周期
</center>

---
title: asio_multithread
categories: essay
comments: false
abbrlink: aef5c1c6
date: 2020-02-01 10:50:43
updated: 2020-02-01 10:50:43
tags:
keywords:
description:
---

# 简介
boost.asio是boost中的一个基于事件的网络库。本文将介绍asio的多线程模型。
asio有两种支持多线程的方案：方案一，开启一个线程池，每个线程独占一个io_context，并在各自的线程中运行io_context的run方法；方案二，开启一个线程池，并创建一个全局的io_context，在每个线程中调用io_context的run方法。
备注：新版本的asio使用io_context代替io_servvice。

# 多io_context方案
在这个多线程方案中，每个线程拥有一个io_context对象，同一个socket不会在多线程中共享，因此不需要引入同步机制。针对io型服务来说，io_context的数量应与cpu数量保持一致；针对计算型服务，请求阻塞了当前线程，当前线程将无法处理其他事件。

## 实现
接着我们来讲一讲这个方案的实现。
为了实现线程池中每个线程拥有一个io_context对象，我们需要先实现一个context池，然后供线程池和其他操作使用。幸运的是，asio代码库的example中提供了这样一个[例子](https://github.com/boostorg/asio/tree/develop/example/cpp03/http/server2)。io_context_pool类即是上文提到的context池，其申明如下:

```
class io_context_pool : private boost::noncopyable {
public:
  explicit io_context_pool(std::size_t pool_size);
  void run();
  void stop();
  boost::asio::io_context& get_io_context();

private:
  typedef boost::shared_ptr<boost::asio::io_context> io_context_ptr;
  typedef boost::asio::executor_work_guard<
    boost::asio::io_context::executor_type> io_context_work;

  std::vector<io_context_ptr> io_contexts_;

  std::list<io_context_work> work_;

  std::size_t next_io_context_;
};
```

这个类中提供了三个方法：
- run方法，创建线程池并在每个线程中运行io_context的run方法
- stop方法，停用所有io_context
- get_io_context方法，使用roundrobin算法获取io_context对象

接着io_context_pool类对象作为server类的成员变量，在start_accept函数和构造函数中使用。server类的声明如下: 

```
class server : private boost::noncopyable {
public:
  explicit server(const std::string& address, const std::string& port, const std::string& doc_root, std::size_t io_context_pool_size);
  void run();

private:
  void start_accept();
  void handle_accept(const boost::system::error_code& e);
  void handle_stop();
  io_context_pool io_context_pool_;
  boost::asio::signal_set signals_;
  boost::asio::ip::tcp::acceptor acceptor_;
  connection_ptr new_connection_;
  request_handler request_handler_;
};
```

# 共享io_context方案
这种方案先创建一个全局的io_context对象，然后开启线程池，在每个线程中调用io_context的run方法。当出现异步事件时，io_context对象会将事件句柄交付给任意线程进行处理。这时io_context不会被某个事件阻塞，但多个线程共享事件循环可能导致socket描述符被多个线程共享，引起竞态条件，为此需要使用asio提供的strand方法来解决io问题。

## 实现
这个方案实现起来相对简单，不再需要实现context池，只需要实现一个线程池，并在线程池中执行io_context的run方法即可。在asio库中包含了这个[例子](https://github.com/boostorg/asio/tree/develop/example/cpp03/http/server3)，在线程池中运行全局的io_context的run方法。

server类的声明如下:

```
class server : private boost::noncopyable {
public:
  explicit server(const std::string& address, const std::string& port, const std::string& doc_root, std::size_t thread_pool_size);
  void run();

private:
  void start_accept();
  void handle_accept(const boost::system::error_code& e);
  void handle_stop();
  
  std::size_t thread_pool_size_;
  boost::asio::io_context io_context_;
  boost::asio::signal_set signals_;
  boost::asio::ip::tcp::acceptor acceptor_;
  connection_ptr new_connection_;
  request_handler request_handler_;
};
```
从server类的声明中可以看到，全局的io_context对象存储在io_context_变量中，并在run函数和构造函数中进行使用。




# 参考
- [The Boost C++ Libraries Chapter 32. Boost.Asio](The Boost C++ Libraries Chapter 32. Boost.Asio)
- [A guide to getting started with boost::asio](https://www.gamedev.net/blogs/entry/2249317-a-guide-to-getting-started-with-boostasio/)
- [Strands: Use Threads Without Explicit Locking](http://www.boost.org/doc/libs/1_59_0/doc/html/boost_asio/overview/core/strands.html)
- [Post on ASIO strand](http://thisthread.blogspot.com/2012/04/post-on-asio-strand.html)
- [How strands guarantee correct execution of pending events in boost.asio](https://stackoverflow.com/questions/39097644/how-strands-guarantee-correct-execution-of-pending-events-in-boost-asio)
- [asio C++ library](https://sourceforge.net/p/asio/mailman/message/19485596/)
[Using Asio with C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3388.pdf)

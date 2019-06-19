---
layout: post
title: Undertow,Tomcat和Jetty服务器配置详解与性能测试
category: other
tags: [other]
---



**undertow**,**jetty**和**tomcat**是javaweb项目当下最火的三款服务器，tomcat是一个老牌的web服务器，是apache下的一款重量级服务器，但是现在微服务盛行，如何选择一款轻量级并且性能优越的服务器是必要的选择。spring-boot 完美集成了tomcat，jetty，undertow，本文重点对比一下各服务器的性能分析及测试。


## 说明
jetty 和 undertow 都是基于NIO实现的高并发轻量级服务器，支持servlet3.1 和 websocket。

NIO 是非阻塞式输入输出
- Channel
- Selector
- Buffer
- Acceptor
   Client和Server只向Buffer读写数据不关注数据的流向，数据通过Channel通道进行流转。而Selector是存在与服务端的，用于Channel的注册以此实现数据I/O操作。Acceptor负责接受所以的连接通道并且注册到Channel中。而整个过程客户端与服务端是非阻塞的也就是异步操作。


###Jetty和Undertow主要配置

​	对于服务器端而言我们关心的重点不是连接超时时间，socket超时时间以及任务的超时时间等的配置，重点是线程池设置，包括工作线程，I/O线程的分配。Jetty在这方面似乎有点太随意，全局使用一个线程池QueuedThreadPool,而最小线程数8最大200，Acceptor线程默认1个，Selector线程数默认2个。而Undertow就比较合理点，Acceptor通过递归循环注册，而用于I/O的线程默认是cpu的线程数，而工作线程是cpu线程数*8。

    **对于服务器而言，如何分配线程可以提高服务器的并发性能。所以，下面将分析两款服务器的详细配置。**

**服务器如何实现通道的注册**

　　Jetty可以设置acceptors的线程数默认是1个。详细实现如下:


```java
    protected void doStart() throws Exception {
        if(this._defaultProtocol == null) {
            throw new IllegalStateException("No default protocol for " + this);
        } else {
            this._defaultConnectionFactory = this.getConnectionFactory(this._defaultProtocol);
            if(this._defaultConnectionFactory == null) {
                throw new IllegalStateException("No protocol factory for default protocol \'" + this._defaultProtocol + "\' in " + this);
            } else {
                SslConnectionFactory ssl = (SslConnectionFactory)this.getConnectionFactory(SslConnectionFactory.class);
                if(ssl != null) {
                    String i = ssl.getNextProtocol();
                    ConnectionFactory a = this.getConnectionFactory(i);
                    if(a == null) {
                        throw new IllegalStateException("No protocol factory for SSL next protocol: \'" + i + "\' in " + this);
                    }
                }

                super.doStart();
                this._stopping = new CountDownLatch(this._acceptors.length);

                for(int var4 = 0; var4 < this._acceptors.length; ++var4) {
                    AbstractConnector.Acceptor var5 = new AbstractConnector.Acceptor(var4, null);
                    this.addBean(var5);
                    this.getExecutor().execute(var5);
                }

 　　　　　　　　this.LOG.info("Started {}", new Object[]{this});
            }
        }
    }
```



 以下是线程详细执行过程。


```java
 public void run() {
            Thread thread = Thread.currentThread();
            String name = thread.getName();
            this._name = String.format("%s-acceptor-%d@%x-%s", new Object[]{name, Integer.valueOf(this._id), Integer.valueOf(this.hashCode()), AbstractConnector.this.toString()});
            thread.setName(this._name);
            int priority = thread.getPriority();
            if(AbstractConnector.this._acceptorPriorityDelta != 0) {
                thread.setPriority(Math.max(1, Math.min(10, priority + AbstractConnector.this._acceptorPriorityDelta)));
            }

            AbstractConnector stopping = AbstractConnector.this;
            synchronized(AbstractConnector.this) {
                AbstractConnector.this._acceptors[this._id] = thread;
            }

            while(true) {
                boolean var24 = false;

                try {
                    var24 = true;
                    if(!AbstractConnector.this.isRunning()) {
                        var24 = false;
                        break;
                    }

                    try {
                        Lock stopping2 = AbstractConnector.this._locker.lock();
                        Throwable var5 = null;

                        try {
                            if(!AbstractConnector.this._accepting && AbstractConnector.this.isRunning()) {
                                AbstractConnector.this._setAccepting.await();
                                continue;
                            }
                        } catch (Throwable var41) {
                            var5 = var41;
                            throw var41;
                        } finally {
                            if(stopping2 != null) {
                                if(var5 != null) {
                                    try {
                                        stopping2.close();
                                    } catch (Throwable var38) {
                                        var5.addSuppressed(var38);
                                    }
                                } else {
                                    stopping2.close();
                                }
                            }

                        }
                    } catch (InterruptedException var43) {
                        continue;
                    }

                    try {
                        AbstractConnector.this.accept(this._id);
                    } catch (Throwable var40) {
                        if(!AbstractConnector.this.handleAcceptFailure(var40)) {
                            var24 = false;
                            break;
                        }
                    }
                } finally {
                    if(var24) {
                        thread.setName(name);
                        if(AbstractConnector.this._acceptorPriorityDelta != 0) {
                            thread.setPriority(priority);
                        }

                        AbstractConnector stopping1 = AbstractConnector.this;
                        synchronized(AbstractConnector.this) {
                            AbstractConnector.this._acceptors[this._id] = null;
                        }

                        CountDownLatch stopping4 = AbstractConnector.this._stopping;
                        if(stopping4 != null) {
                            stopping4.countDown();
                        }

                    }
                }
            }

            thread.setName(name);
            if(AbstractConnector.this._acceptorPriorityDelta != 0) {
                thread.setPriority(priority);
            }

            stopping = AbstractConnector.this;
            synchronized(AbstractConnector.this) {
                AbstractConnector.this._acceptors[this._id] = null;
            }

            CountDownLatch stopping3 = AbstractConnector.this._stopping;
            if(stopping3 != null) {
                stopping3.countDown();
            }

        }
```



　　可以看到通过while循环监听所有建立的连接通道，然后在将通道submit到SelectorManager中。

 Undertow就没有这方面的处理，通过向通道中注册Selector，使用ChannelListener API进行事件通知。在创建Channel时，就赋予I/O线程，用于执行所有的ChannelListener回调方法。 



```
SelectionKey registerChannel(AbstractSelectableChannel channel) throws ClosedChannelException {
    if(currentThread() == this) {
        return channel.register(this.selector, 0);
    } else if(THREAD_SAFE_SELECTION_KEYS) {
        SelectionKey task1;
        try {
            task1 = channel.register(this.selector, 0);
        } finally {
            if(this.polling) {
                this.selector.wakeup();
            }

        }

        return task1;
    } else {
        WorkerThread.SynchTask task = new WorkerThread.SynchTask();
        this.queueTask(task);

        SelectionKey var3;
        try {
            this.selector.wakeup();
            var3 = channel.register(this.selector, 0);
        } finally {
            task.done();
        }

        return var3;
}
```



　　所以:无论设计架构如何可以看到两个服务器都是基于NIO实现的，而且都有通过Selector来执行所有的I/O操作，通过IP的hash来将Channel放入不同的WorkThread或SelectorManager中，然后具体的处理工作有线程池来完成。所有，我个人认为Jetty的selectors数和Undertow的IOThreads数都是用于Selector或说是做I/O操作的线程数。不同的是Jetty全局线程池。而对于两个服务器的承载能力以及读写效率，包括LifeCycle过程的管理等，决定了两个服务器性能的好坏。毕竟用于工作的线程所有的开销在于业务，所有个人觉得：I/O操作，管理与监听，决定了两个服务器的优劣。

### Jetty和Undertow压测分析

**准备工具：**

- **siege用于压测**


- **VisualVm用于监测**

**项目准备:**

   Jetty：acceptors=1，selectors=2, min and max threads=200 

   Undertow: work_threads=200,io_threads=2

**压测梯度:**

siege -c 50 -r 2000 -t 2 --log=./result.log http://127.0.0.1:8080/test

siege -c 100 -r 2000 -t 2 --log=./result.log http://127.0.0.1:8080/test

siege -c 200 -r 2000 -t 2 --log=./result.log http://127.0.0.1:8080/test

**测试结果:**

| 服务器      | 命中    | 成功率  | 吞吐量              | 平均耗时    |
| -------- | :---- | ---- | ---------------- | ------- |
| Jetty    | 11736 | 100% | 98.13 trans/sec  | 0.00sec |
|          | 23182 | 100% | 193.36 trans/sec | 0.01sec |
|          | 47098 | 100% | 393.66 trans/sec | 0.01sec |
| Undertow | 12062 | 100% | 100.52 trans/sec | 0.00sec |
|          | 23563 | 100% | 197.59 trans/sec | 0.01sec |
|          | 47290 | 100% | 394.08 tran/sec  | 0.01sec |
| Tomcat   | 11674 | 100% | 97.98 trans/sec  | 0.01sec |
|          | 23796 | 100% | 198.76 trans/sec | 0.01sec |
|          | 47131 | 100% | 393.09 trans/se  | 0.01sec |

从中可以看出在高负载下Undertow的吞吐量高于Jetty而且随着压力增大Jetty和Undertow成功率差距会拉大。而在负载不是太大情况下服务器处理能力差不多，jetty还略微高于Undertow。而tomcat的负载能力似乎和Undertow很接近。

对比三个服务器发现在Undertow在负载过重情况下比Jetty和Tocmat更加顽强，实践证明在负载继续加大情况下Undertow的成功率高于其它两者，但是在并发不是太大情况下三款服务器整体来看差别不大。此次测试网络传输数据量太小，所以没有通过不断加大数据传输量来观察负载情况，个人r认为测试一款服务器的I/O情况，还要通过改变数据传输量来看看在大数据文本高负载下三款服务器的性能。

### 大数据量测试

使用1892byte回复数据测试三款服务器性能，下面是开启线程执行情况图。

Undertow:

  ![img](https://ldmyown.github.io\assets\images\2019\web\1.png)

Jetty:

![img](https://ldmyown.github.io\assets\images\2019\web\2.png)

Tomcat:

![img](https://ldmyown.github.io\assets\images\2019\web\3.png)

　　实验过程发现Undertow和Tomcat的负载能力很接近但是Undertow比较好点，而Jetty远远不足。通过观察以上三张图不难发现，Undertow的I/O线程执行100% , Tomcat的执行也是100%两者不同的是Undertow用于I/O的线程数是可以调整的，而Tomcat不可以，起码通过spring boot 无法调整，这样就制约了它的负载能力。而Jetty由于全局共享线程池所以，会存在Selector和Acceptor阻塞情况，这样就制约了I/O操作。但是有个好处就是在负载不是太重的情况下可以使工作线程有更多占用资源来处理程序，提高了吞吐量。但是，总体而言这种差距是很小的。
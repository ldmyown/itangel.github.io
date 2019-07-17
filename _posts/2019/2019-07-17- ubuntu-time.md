---
layout: post
title: Linux下使用timedatectl命令时间时区操作详解
category: other
tags: [other]
---



# 

　　timedatectl命令对于RHEL / CentOS 7和基于Fedora 21+的分布式系统来说，是一个新工具，它作为systemd系统和服务管理器的一部分，代替旧的传统的用在基于Linux分布式系统的sysvinit守护进程的date命令。

　　timedatectl命令可以查询和更改系统时钟和设置，你可以使用此命令来设置或更改当前的日期，时间和时区，或实现与远程NTP服务器的自动系统时钟同步。

　　在本教程中，我要讲的是，如何在你的Linux系统上，通过使用来自于终端使用timedatectl命令的NTP，设置date、time、timezone和synchronize time来管理时间。让你的Linux服务器或系统保持正确的时间是一个很好的实践，它有以下优点：

　　1）维护及时操作的系统任务，因为在Linux中的大多数任务都是由时间来控制的。

　　2）记录事件和系统上其它信息等的正确时间。

 

**如何查找和设置Linux本地时区**

1、要显示系统的当前时间和日期，使用命令行中的timedatectl命令，如下：

```
# timedatectl status
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113125840760-1412416063.gif)

　　在上面的示例中，RTC time就是硬件时钟的时间。

2、Linux系统上的time总是通过系统上的timezone设置的，要查看当前时区，按如下做：

```
# timedatectl 
OR
# timedatectl | grep Time
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130255760-443096740.gif)

3、要查看所有可用的时区，运行以下命令：

```
# timedatectl list-timezones
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130353681-1064142284.gif)

4、要根据地理位置找到本地的时区，运行以下命令：

```
# timedatectl list-timezones | egrep -o ‘’Asia/B.*”
# timedatectl list-timezones | egrep -o “Europe/L.*”
# timedatectl list-timezones | egrep -o “America/N.*”
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130554931-355176850.gif)

5、要在Linux中设置本地时区，使用set-timezone开关，如下所示。

```
# timedatectl set-timezone "Asia/Kolkata"
```

　　中国上海的时区：

```
# timedatectl set-timezone "Asia/Shanghai"
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130723338-1706546881.gif)

 　　推荐使用和设置协调世界时，即UTC。

```
# timedatectl set-timezone UTC
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130915228-1721727285.gif)

 　　你需要输入正确命名的时区，否者在你改变时区的时候，可能会发生错误。在下面的例子中，由于 “Asia/Kalkata” 这个时区是不正确的，因此导致了错误。

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131009728-29191656.gif)

 

**如何在Linux中设置时间和日期**

　　你可以使用timedatectl命令，设置系统上的日期和时间，如下所示：

6、设置Linux中的时间。只设置时间的话，我们可以使用set-time开关以及HH：MM：SS（小时，分，秒）的时间格式。

```
# timedatectl set-time 15:58:30
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131240260-1561897221.gif)

 7、在Linux中设置日期。只设置日期的话，我们可以使用set-time开关以及YY：MM：DD（年，月，日）的日期格式。

```
# timedatectl set-time 20151120
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131335525-1442987888.gif)

8、设置日期和时间：

```
# timedatectl set-time '16:10:40 2015-11-20'
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131704369-1771813681.gif)

 

**如何在Linux中查找和设置硬件时钟**

------

 

9、要设置硬件时钟以协调世界时，UTC，可以使用 set-local-rtc boolean-value选项，如下所示：

　　首先确定你的硬件时钟是否设置为本地时区：

```
# timedatectl | grep local
```

　　将你的硬件时钟设置为本地时区：

```
# timedatectl set-local-rtc 1
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113132023681-700633329.gif)

　　将你的硬件时钟设置为协调世界时（UTC）：

```
# timedatectl set-local-rtc 0
```

![img](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131936135-1877678773.gif)

 

**将Linux系统时钟同步到远程NTP服务器**

　　NTP即Network Time Protocol（网络时间协议），是一个互联网协议，用于同步计算机之间的系统时钟。timedatectl实用程序可以自动同步你的Linux系统时钟到使用NTP的远程服务器。

　　注意，你必须在系统上安装NTP以实现与NTP服务器的自动时间同步。

　　要开始自动时间同步到远程NTP服务器，在终端键入以下命令。

```
# timedatectl set-ntp true
```

　　要禁用NTP时间同步，在终端键入以下命令。

```
# timedatectl set-ntp false
```



---
layout: post
title: windows 下挂载nfs目录
category: windows
tags: [linux]
---



## 如何在windows下挂载nfs目录

此处以win10为例，讲解一下如何在windows中挂载nfs目录

### 安装nfs客户端

- 打开win10控制面板，在搜索框中输入控制面板，可以快速定位到控制面板

  ![1571197964151](https://ldmyown.github.io\assets\images\2019\nfs\1571197964151.png)

- 选择程序进入

  ![1571198074540](https://ldmyown.github.io\assets\images\2019\nfs\1571198074540.png)

- 打开**启动或关闭windows功能**菜单

  ![1571198142968](https://ldmyown.github.io\assets\images\2019\nfs\1571198142968.png)

- 勾选上**适用于Linux的windows子系统**，展开NFS服务，勾选**NFS客户端**和**管理工具**然后点确定，等待安装完成，安装完成后重启系统。

  ![1571198380901](https://ldmyown.github.io\assets\images\2019\nfs\1571198380901.png)

### 检测安装结果

- 在win10 中打开cmd命令提示符窗口，输入mount -h，查看运行结果，结果如下则表示安装成功。

  ![1571204167468](https://ldmyown.github.io\assets\images\2019\nfs\1571204167468.png)



### 挂载目录

使用命令**mount \\NFS的IP地址或者主机名\nfs目录名 挂载点**挂载目录

我的nfs服务器为**192.168.174.130**目录为**nfsroot**，则挂载命令为：

```
mount \\192.168.174.130\nfsroot x:
```

![1571204790546](https://ldmyown.github.io\assets\images\2019\nfs\1571204790546.png)



在资源管理器中查看已经存在对应的磁盘，在磁盘中可以操作对应的文件

![1571204836343](https://ldmyown.github.io\assets\images\2019\nfs\1571204836343.png)

### 取消挂载

取消挂载命令：``umount  挂载点``

例如： umount x:  ,就可以取消磁盘的挂载了。
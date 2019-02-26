---
layout: post
title: 精简版mysql安装教程
category: other
tags: [other]
---





在windows中使用mysql原版安装包，会附带安装很多不需要的软件，本文介绍一下如何精简安装mysql。



## 下载mysql

到官网下载Community 版的MySQL Community Server ：https://dev.mysql.com/downloads/

##  提取需要的文件
下载好后，解压以下文件到新文件夹，例如mysql

- bin/mysql.exe
- bin/mysqld.exe
- bin/mysqladmin.exe
- share/charsets
- share/english

在新文件夹（mysql）中新建my.ini 
粘贴以下内容
~~~
[mysql]  
# 设置mysql客户端默认字符集  
default-character-set=utf8  
[mysqld]  
#设置3306端口  
port = 3306  
# 设置mysql的安装目录  
basedir=D:\mysql\mysql-5.6.17-winx64  
# 设置mysql数据库的数据的存放目录  
datadir=D:\mysql\mysql-5.6.17-winx64\data  
# 允许最大连接数  
max_connections=200  
# 服务端使用的字符集默认为8比特编码的latin1字符集  
character-set-server=utf8  
# 创建新表时将使用的默认存储引擎  
default-storage-engine=INNODB  
~~~

## 安装

- 以管理员身份运行cmd
- cd 进bin目录
- 执行初始化数据 mysqld --initialize-insecure
- 执行服务安装 mysqld install

- 然后启动mysql服务   net start mysql
- 停止mysql服务   net stop mysql

- 然后通过mysql.exe 来检测是否成功运行mysql服务   mysql -u root -p

## 设置root密码
- 格式：mysql> set password for 用户名@localhost = password('新密码'); 
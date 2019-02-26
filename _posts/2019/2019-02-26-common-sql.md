---
layout: post
title: 常用数据库脚本（持续更新中）
category: other
tags: [other]
---





本文将记录在使用数据库过程中，经常使用的数据库语句，以及数据库使用过程中可能遇到的一些问题解决方案



## mysql修改root用户的密码

### 用SET PASSWORD命令 
首先需要使用root的用户名密码登陆到mysql中。 

~~~mysql
格式：set password for 用户名@localhost = password('新密码'); 
例子：set password for root@localhost = password('123'); 
~~~



### 用mysqladmin 
格式：mysqladmin -u用户名 -p旧密码 password 新密码 
例子：mysqladmin -uroot -p123456 password 123 



### 用UPDATE直接编辑user表 

首先需要使用root的用户名密码登陆到mysql中。 

~~~mysql
use mysql; 

格式：update user set password=password('新密码') where user='root' and host='localhost';

例子： update user set password=password('123') where user='root' and host='localhost';

flush privileges; 

~~~

# mysql修改用户远程访问权限

首先需要使用root的用户名密码登陆到mysql中。 

~~~mysql
grant all privileges  on  *.* to root@'%' identified by "password";

flush privileges;

~~~





## sql server 中删除数据库

在sql server中删除数据库时，经常会提示数据库正在使用无法删除，那么我们可以通过一下命令来删除不需要的数据库。

~~~mssql
use master;
alter database "audit-620" set single_user with rollback immediate ;
drop database "audit-620";
~~~


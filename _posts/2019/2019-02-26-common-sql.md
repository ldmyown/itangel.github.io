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



## mysql 5.7 修改用户的密码

~~~
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '';
~~~



## mysql 修改隔离事务级别

查看MySQL的隔离事务级别，

查看InnoDB存储引擎 系统级的隔离级别 和 会话级的隔离级别：

~~~
mysql> select @@global.tx_isolation,@@tx_isolation;
+-----------------------+-----------------+
| @@global.tx_isolation | @@tx_isolation  |
+-----------------------+-----------------+
| REPEATABLE-READ       | REPEATABLE-READ |
+-----------------------+-----------------+
1 row in set (0.00 sec)
~~~

修改mysql的隔离事务级别

~~~
设置innodb的事务级别方法是：set 作用域 transaction isolation level 事务隔离级别，例如~

SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}

mysql> set global transaction isolation level read committed; //全局的

mysql> set session transaction isolation level read committed; //当前会话
~~~




**Read Uncommitted（读取未提交内容）**

       在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。
**Read Committed（读取提交内容）**

       这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。
**Repeatable Read（可重读）**

       这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。

**Serializable（可串行化）** 
       这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

         这四种隔离级别采取不同的锁类型来实现，若读取的是同一个数据的话，就容易发生问题。例如：

         脏读(Drity Read)：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。

         不可重复读(Non-repeatable read):在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。

         幻读(Phantom Read):在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。

         在MySQL中，实现了这四种隔离级别，分别有可能产生问题如下所示：

![img](https://images2015.cnblogs.com/blog/156233/201604/156233-20160425233944502-1414059978.jpg)
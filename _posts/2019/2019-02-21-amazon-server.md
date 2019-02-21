---
layout: post
title: 亚马逊免费云服务器搭建
category: other
tags: [other]
---

# 亚马逊云（EC2）搭建教程，可免费使用一年

​	最近想搭一个vps，查询到亚马逊云服务器能够免费使用一年，然后就查询了相关的资料，整理了这篇文章记录一下免费的服务器的搭建方法。

​	亚马逊云，可以免费使用一年，不过每个月在使用上也有一些限制，比如流量每个月只能免费使用 15 G的出口流量，超出就要额外收费，具体免费的限制可以看官网这篇文章：https://amazonaws-china.com/cn/free/faqs/ 。

## aws 注册前提条件

1. 要有一个亚马逊账号
2. 需要有一张双币信用卡，比如 Visa、MasterCard、用于注册验证。

## 亚马逊云注册步骤

官网地址：https://amazonaws-china.com/cn

- 如果你没有亚马逊账号，那么先注册一个账号，访问官网后，点击右上角的注册选项开始注册：



![选择注册选项，开始注册亚马逊账号](https://ldmyown.github.io\assets\images\2019\amazon-server\aws1.png)

- 输入邮箱、密码等信息，如：

![输入邮箱等信息](https://ldmyown.github.io\assets\images\2019\amazon-server\aws2.png)

- 填写联系人信息，如下：

![填写账号类型](https://ldmyown.github.io\assets\images\2019\amazon-server\aws3.png)

![填写个人信息](https://ldmyown.github.io\assets\images\2019\amazon-server\aws4.png)

- 点击创建账户并继续，如果上面填写的信息没问题，则进入到信用卡验证这一步。如下：

![验证个人信息及信用卡信息](https://ldmyown.github.io\assets\images\2019\amazon-server\aws5.png)

- 信息提交成功后，信用卡会扣款 1 美元。接着进入下一步，电话验证：

![输入电话号码，获取 pin 码](https://ldmyown.github.io\assets\images\2019\amazon-server\aws6.png)

点击立即呼叫我，然后屏幕上就会显示 4 位 pin 码，紧接着你会收到来自亚马逊的验证电话：

![电话拨通成功后，屏幕显示 pin 码](https://ldmyown.github.io\assets\images\2019\amazon-server\aws7.png)

接听电话后，你在电话正确输入屏幕上显示的 4 位 pin 码，即可验证成功：

![手机输入pin码，验证成功提示](https://ldmyown.github.io\assets\images\2019\amazon-server\aws8.png)

- 接着点击继续进入下一步，选择支持计划：

![选择免费方案](https://ldmyown.github.io\assets\images\2019\amazon-server\aws9.png)

- 这里我只是免费体验一下，所以选择的是免费的方案，服务器性能较低，如果是土豪可以选择收费套餐


- 下一步，会告诉我们账户正在激活中，如果成功激活，则会向我们绑定的邮箱中发一封激活邮件，这个过程大概几分钟的时间，此过程中不能操作创建实例，请耐心等待

![成功后会收到一封邮件提示](https://ldmyown.github.io\assets\images\2019\amazon-server\aws10.png)

## 亚马逊云 EC2 搭建

- 点击右上角的我的账户，然后选择 AWS 控制台，输入账号、密码进行登录：

![登录 aws 控制台](https://ldmyown.github.io\assets\images\2019\amazon-server\aws11.png)

- 登录后，右上角的地区选择一个合适的地区，如：

![选择合适的地区](https://ldmyown.github.io\assets\images\2019\amazon-server\aws12.png)

- 左上角的服务，选择 EC2，如：

![在控制台选择 EC2](https://ldmyown.github.io\assets\images\2019\amazon-server\aws13.png)

- 在新开的页面中，会有一个创建实例的选项：

![创建EC2实例](https://ldmyown.github.io\assets\images\2019\amazon-server\aws14.png)

- 点击启动实例，进入到选择一个 Amazon 系统映像选项，往下拉，找到 Ubuntu Server 14.04 LTS (HVM) 选项（注意，要记得选择符合条件的免费套餐）：

![选择免费配置套餐](https://ldmyown.github.io\assets\images\2019\amazon-server\aws15.png)

- 点击选择选项，进入到实例类型选项，仍旧选择符合条件的免费套餐，然后点击审核和启动：

![审核和启动](https://ldmyown.github.io\assets\images\2019\amazon-server\aws16.png)

- 核查实例启动，这里不用操作什么，直接点击启动：

![启动创建实例](https://ldmyown.github.io\assets\images\2019\amazon-server\aws17.png)

然后出现一个选择现有密钥对或创建新密钥对的弹窗，这里我们选择创建新密钥对，然后密钥对名称你可以随便取一个名字，不过该名字在登录时有用，这里我取lid , 如下：

![输入密钥对名称，下载密钥对](https://ldmyown.github.io\assets\images\2019\amazon-server\aws18.png)

然后点击下载密钥对，下载一个后缀为 pem 的文件,因为我这里名称为lid，所以下载的文件名称为 lid.pem 。(注意该文件记得保存，后面会用到，用于服务器登录)。然后启动实例即可：

![选择密钥对，然后启动实例](https://ldmyown.github.io\assets\images\2019\amazon-server\aws19.png)

启动后，会提示启动状态：

![启动成功后，显示启动状态](https://ldmyown.github.io\assets\images\2019\amazon-server\aws20.png)

如果你害怕不小心超出免费账单，那么可以选择创建账单警报。这里我忽略。

过一会而重新返回 EC2 首页，就会看到实例已经启动并在运行了：

![返回首页，显示实例正在运行](https://ldmyown.github.io\assets\images\2019\amazon-server\aws23.png)

点击正在运行的实例，可以查看外网 ip 、共有 dns 等信息：

![获取ip等实例信息](https://ldmyown.github.io\assets\images\2019\amazon-server\aws24.png)

## 使用 xshell 连接 EC2 服务器

这里连接工具我选择 xshell，你也可以选择其它连接工具。打开 xshell，然后在工具选项选择用户密钥管理者：

![xshell选择用户密钥管理者](https://ldmyown.github.io\assets\images\2019\amazon-server\aws21.png)

选择后，在弹出的对话框里面导入之前保存的 pem 文件，如下;

![导入pem文件](https://ldmyown.github.io\assets\images\2019\amazon-server\aws22.png)

导入后，新建连接，如：

![创建新连接](https://ldmyown.github.io\assets\images\2019\amazon-server\aws25.png)

然后在新建连接里面输入名称（随便填）、协议 ssh、主机（EC2 实例的连接ip)、端口号 22 ：

![输入端口、ip、协议等信息](https://ldmyown.github.io\assets\images\2019\amazon-server\aws26.png)

信息填完后，不要点击确定，点击连接下方的用户身份验证：

![进行身份验证](https://ldmyown.github.io\assets\images\2019\amazon-server\aws27.png)

注意：方法选择 Public key，用户名填 ubuntu （根据你选择的 EC2 实例类型），用户密钥选择之前导入的 pem 文件。官方文档是说：
每个 Linux 实例类型均使用默认 Linux 系统用户账户启动。 
对于 Amazon Linux，用户名称是 ec2-user。
对于 RHEL5，用户名称是 root 或 ec2-user。
对于 Ubuntu，用户名称是 ubuntu。
对于 Fedora，用户名称是 fedora或 ec2-user。
对于 SUSE Linux，用户名称是 root 或 ec2-user。

你可以根据你自己选择的系统类型选择填写对应的用户名。

只要信息没填写错误，点击确定后就可以连接成功：

![成功连接到 EC2](https://ldmyown.github.io\assets\images\2019\amazon-server\aws28.png)

然后就可以开始搭建各种服务了。:) 。如果在搭建过程中需要使用 root 用户，使用 sudo -i 命令切换到 root 用户即可。上面关于亚马逊云aws搭建基本就完成了，不过还要记得配置防火墙。

## 修改 EC2 防火墙配置

在搭建完相应的服务后，发现始终连接不上。亚马逊云 EC2 防火墙入站配置默认只开启了 22 端口的流量，你可以修改该配置，把协议和端口范围都改成全部，或者新建一条防火墙规则，开启某个特定端口。

在实例列表中，往右边拖动，有一个安全组标签，选择该标签进入防火墙配置：

![在实例列表找到安全组标签](https://ldmyown.github.io\assets\images\2019\amazon-server\aws29.png)

然后编辑入站规则：

![编辑入站规则](https://ldmyown.github.io\assets\images\2019\amazon-server\aws30.png)

改成所有流量通行，或者你可以添加其它端口规则：

![修改入站流量规则](https://ldmyown.github.io\assets\images\2019\amazon-server\aws31.png)

修改完后，基本上就立即生效了，这时候就可以愉快的使用各种服务了。


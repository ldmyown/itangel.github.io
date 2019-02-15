---
layout: post
title: idea安装插件使用代理
category: idea
tags: [idea]
---

## 说明
作为一个java 程序猿，几乎都知道Intellij IDEA这一款神器，他的界面清爽，功能强大，可使用的插件多彩多样。

刚开始做java开发的时候，使用的eclipse 开发工具，后来经由同事推荐后，使用了idea这款工具，使用后发现他太好用了，就抛弃了eclipse，

开始使用idea，但是在下载插件的时候，由于某些插件连接的是国外的网站，导致下载速度慢或者直接下载失败。

后来研究了下，可以使用代理的方式科学上网来下载这些插件，于是把设置的方式记录下来。

## 操作步骤
- 前提：已经搭建科学上网软件

- 在插件下载页面点击Http Proxy Settings打开代理设置

   ![idea_proxy](https://ldmyown.github.io\assets\images\2019\idea\idea_proxy.png)

- 在代理设置页面中设置所搭建的科学上网服务地址与端口

   ![idea_proxy_2](https://ldmyown.github.io\assets\images\2019\idea\idea_proxy_2.png)


- 点Check connection 能够连接上则表示配置成功。
- 然后就可以快速的下载所需要的插件了。
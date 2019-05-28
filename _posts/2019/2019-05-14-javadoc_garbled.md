---
layout: post
title: idea 生成 javadoc乱码问题解决
category: other
tags: [other]
---

### 说明
在使用IDEA生成Java Doc的过程中，发现IDEA控制台乱码，于是寻找了一下解决方法。

### 解决方法
- 添加maven打包参数

​	在IDEA中，打开File | Settings | Build, Execution, Deployment | Build Tools | Maven | Runner在VM Options中添加**-Dfile.encoding=GBK**，因为Maven的默认平台编码是GBK，所以这里需要使用GBK的编码才不会乱码，如果你在命令行中输入mvn -version的话，会得到如下信息，根据Default locale可以看出

~~~
Maven home:…
Java version:…
Java home:…
Default locale: zh_CN, platform encoding: GBK
…
…
~~~





- 再次运行mvn clean package，控制台输出就不会乱码了。

  ​




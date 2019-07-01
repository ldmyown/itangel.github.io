---
layout: post
title: 通过linux镜像文件做定制后重新打包成ISO镜像
category: other
tags: [other]
---



# Ubuntu18.04挂载Samba共享文件夹



## 安装cifs-utifs

### 软件安装

~~~
sudo apt-get install cifs-utils
~~~

###  列举指定IP地址所提供的共享文件夹列表

~~~
smbclient -L ${ip_addr} -U ${username}%${password}
~~~



###  挂载共享文件夹

~~~
mount -t cifs ${remount_share_folder}  ${local_mount_folder} -o username=${username},password=${password}
eg. mount -t cifs //192.168.1.1/share /mnt/share -o username=root,password=123456
~~~









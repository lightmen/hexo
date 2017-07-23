---
title: 本地挂载android ext4镜像
categories: 
tags:
- android
- ext4
---

android系统的data.img、system.img等ext4打包的镜像是可以挂载到本地PC环境的，方法如下：
```
$ simg2img system.img system.raw.img
$ sudo mount -t ext4 -o loop system.raw.img ./my_system  
```

首先unpack system.raw.img文件，然后用mount即可。
(注意：如果ubuntu中没有simg2img的话，输入sudo apt-get install android-tools-fsutils安装)

Windows下可以下载一个simg2img_win.rar的包文件



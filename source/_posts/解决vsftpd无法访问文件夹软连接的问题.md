---
title: 解决VSFTPD无法访问文件夹软连接的问题
date: 2018-09-05 11:25:02
categories: [Linux, FTP]
tags: [Linux, FTP]
---
vsftpd本身不支持软连接，而Linux内核从2.4.0开始支持把一部分文件系统挂载到文件系统中的其他位置，`mount`命令的`--bind`选项正好提供了这个功能。
<!--more-->
通过命令
```
mount --bind /path/to/share /path/of/ftp/subpath
```
可以把需要共享的文件夹`/path/to/share`挂载到FTP目录中的一个子目录上`/path/of/ftp/subpath`。这个目录对于vsftpd而言是一个正常文件系统的目录，于是就可以被共享了。当不需要共享目录时，直接`umount`即可。
`mount --bind`命令本身支持单个文件的挂载，可以把目标文件挂载到另外一个文件上，起到类似于软链接的功能。同目录的挂载类似，这也是vsftpd支持的。

而以上方法，在系统重启后会失去效果，所以需要设置开机自动挂载。
方法是在`/etc/rc.local`中添加相应的指令：
```
mount --bind /path/to/share /path/of/ftp/subpath
```
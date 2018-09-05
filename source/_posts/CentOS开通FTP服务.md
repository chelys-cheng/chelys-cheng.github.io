---
title: CentOS开通FTP服务
date: 2018-05-29 22:56:55
categories: [Linux, FTP]
tags: [Linux, FTP]
---
## FTP工作原理
FTP有两个过程一个是**控制连接**，一个是**数据传输**。需要两个端口，一个端口是作为控制连接端口，也就是FTP的21端口，用于发送指令给服务器以及等待服务器响应；另外一个端口用于数据传输端口，端口号为20（仅用PORT模式），是用建立数据传输通道的，主要作用是从客户向服务器发送一个文件，从服务器向客户发送一个文件，从服务器向客户发送文件或目录列表。
<!--more-->
## FTP连接方式
FTP支持两种连接模式：主动模式（Port）和被动模式（Pasv），这两种模式都是针对**数据链路**进行的，与控制链路无关。

### 主动模式工作过程：
1. 客户端从自己的一个任意端口（N > 1024）和FTP服务器的21端口建立控制链路
1. 然后客户端发出Port指令告诉服务器连接自己的N+1端口来建立一条数据通道
1. 当FTP服务器接到这一指令时，会使用20端口连接用户在Port指令中指定的端口号N+1来发送数据

### 被动模式工作过程：
1. 客户端从自己的一个任意端口（N > 1024）和FTP服务器的21端口建立控制链路
2. 然后客户端发送Pasv指令，告诉服务器自己要连接服务器的某一个端口
3. 如果服务器上的这个端口是空闲可用的，那么服务器会返回确认信息，之后数据传输通道被建立；但如果服务器上的这个端口被另一个资源所使用，那么服务器返回不确认的信息，那么这是客户端会再次发送Pasv命令。

**注意**：

    在FTP客户连接服务器的整个过程中，控制信道是一直保持连接的，而数据传输通道是临时建立的；
    主动模式 建立数据传输通道是由服务器端发起的，服务器使用20端口连接客户端某一个大于1024的端口；
    被动模式 建立数据传输通道是由客户端发起的，它使用一个大于1024的端口连接服务器的1024端口以上的某一个端口。

## 搭建FTP服务器
在centos7中安装vsftpd,实现FTP服务,并创建两个帐号(非虚拟帐号):
 user | group | home 
------|-------|------
 test1 | ftp | /usr/ftp 
 test2 | ftp | /usr/ftp/test2

### 安装流程

1. 查看系统有没有安装vsftpd
```
$ rpm -q vsftpd
```
2. 安装vsftpd
```
# yum -y install vsftpd
```
3. 启动vsftpd
```
# systemctl start vsftpd
```
4. 创建新用户,并加入到ftp用户组中,并设置密码
```
# useradd test1 -d /usr/ftp -g ftp -s /sbin/nologin
# useradd test2 -d /usr/ftp/test2 -g ftp -s /sbin/nologin
# passwd test1  
# passwd test2 
```
5. 配置文件
```
-rw-------   1 root root   125 8月   3 2017 ftpusers    # 用于指定哪些用户不能访问FTP服务器，固定黑名单，不受配置影响。
-rw-------   1 root root   361 8月   3 2017 user_list   # 指定允许使用vsftpd的用户列表文件。
-rw-------   1 root root  5030 8月   3 2017 vsftpd.conf # vsftpd的核心配置文件。
-rwxr--r--   1 root root   338 8月   3 2017 vsftpd_conf_migrate.sh  # vsftpd 操作的一些变量和设置脚本,可以不用管。
```
我主要设置的项如下
```
anonymous_enable=NO         # 禁止匿名登录
local_enable=YES            # 允许本地用户登录ftp服务器
write_enable=YES            # 允许用户向服务器执行写操作
dirmessage_enable=YES       # 启用目录提示消息
pasv_enable=YES             # 启用被动模式,其实默认就是YES
pasv_min_port=1242          # 被动模式下,使用的数据传输端口最低为1242
pasv_max_port=1243          # 被动模式下,使用的数据传输端口最高为1243
xferlog_std_format=YES      # 启用标准的日志格式
ascii_upload_enable=YES     # 允许上传过程中使用ASCII字符传输
ascii_download_enable=YES   # 允许下载过程中使用ASCII字符传输
chroot_local_user=YES       # 启用将用户限定在主目录下
listen_port=2021            # 修改默认的端口为2021,为了安全起见
userlist_enable=YES         # 两项结合表示只允许userlist中的用户登录使用FTP服务
userlist_deny=NO            # 两项结合表示只允许userlist中的用户登录使用FTP服务
```
**特别的**

    userlist_enable和userlist_deny两个搭配起来，一共四种模式。

    其中，userlist_deny是否生效取决于userlist_enable是否启用。

    userlist_enable=false，表示不启用user_list文件,无论userlist_deny=true或者userlist_deny=false，均无法造成影响。
    此时的模式是除了ftpusers文件中的用户，其他本地用户均可以登录FTP服务器，无视user_list中的名单。

    userlist_enable=true,表示启用user_list文件，此时的user_list是黑名单还是白名单，就取决于userlist_deny的值：
    userlist_deny=NO，则只允许这些用户登录FTP服务器；
    userlist_deny=YES，则禁止这些用户登录FTP服务器。

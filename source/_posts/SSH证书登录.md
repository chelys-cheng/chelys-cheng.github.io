---
title: SSH证书登录
date: 2018-05-29 21:30:20
categories: [Linux, SSH]
tags: [Linux, SSH]
---
ssh有密码登录和证书登录。对于外网的机器，在使用密码登录时，很容易遭到攻击，使用ssh登录还是证书登录比较安全一些。
<!--more-->
# 配置证书的流程
 - 生成证书，包括私钥和公钥，私钥在客户端保存，必要时设置密码，这样每次在登录的时候，都需要输入密码才会揭开私钥。
 - 服务器保存公钥，并配置使用证书登录。

## 生成证书
```
# 默认生成
# id_rsa: 私钥，放在客户端
# id_rsa.pub: 公钥，放在服务器

[root@localhost ~]# ssh-keygen -t rsa
```
## 公钥上传服务器
从客户端将公钥上传到服务器
```
[root@localhost ~]# scp ~/.ssh/id_rsa.pub username@remotehost:~
```
## 服务器端导入公钥
在服务器端，将公钥导入到ssh证书验证默认文件，并设置正确的权限
```
[root@remotehost ~]# cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
# 如果没有正确设置权限，会导致登录失败
[root@remotehost ~]# chown -R 0700  ~/.ssh
[root@remotehost ~]# chown -R 0644  ~/.ssh/authorized_keys
```
## 服务器端配置
修改服务器端的SSH配置，支持证书登录
```
[root@remotehost ~]# vim /etc/ssh/sshd_config
# 主要修改以下配置：
RSAAuthentication yes
StrictModes no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
# 非必要配置，属于加固配置
Port  22222                 # 不使用默认端口22，改用22222
PermitRootLogin no          # 禁止使用root登录
MaxAuthTries 1              # 限制SSH验证重试次数
PasswordAuthentication no   # 禁止使用明文密码登录
```
## 重启服务器
服务器端，重启SSH服务
```
[root@remotehost ~]# systemctl restart sshd
```
## 客户端配置登录信息
```
[root@localhost ~]# touch ~/.ssh/config
# 配置以下内容
Host ssh-server                 # 此配置名称
Hostname remotehost             # 远程服务器地址
Port 22222                      # SSH服务使用的端口
IdentityFile ~/.ssh/id_rsa      # 本地私钥的地址
User username                   # 登录用户名

# 本机管理多个私钥，可以配置多份
# Windows10 的配置文件放在C:\Users\Administrator\.ssh中
```
## 登录SSH服务器
配置后登录SSH服务器命令：
```
[root@localhost ~]# ssh ssh-server
```
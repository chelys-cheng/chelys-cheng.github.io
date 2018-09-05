---
title: 安装npm时候遇到的bug记录
date: 2018-05-27 12:25:11
categories: [Linux, npm]
tags: [JavaScript, npm, Linux]
---
之前在装npm的时候，遇到了一个问题，自己做了一些修改，然后成功安装成功，在此做一下记录。
<!--more-->

安装npm时，我的做法是从淘宝的npm镜像下载源文件，直接解压使用。
```
[me@localhost ~]$ wget https://npm.taobao.org/mirrors/npm/v6.1.0.tar.gz
[me@localhost ~]$ tar xzf v6.1.0.tar.gz
[me@localhost ~]$ cd npm-6.1.0/bin
[me@localhost bin]$ ./npm -v
```
但是出现了错误[/home/me/为个人账户文件夹]：
```
module.js:549
    throw err;
    ^

Error: Cannot find module '/home/me/npm-6.1.0/bin/node_modules/npm/bin/npm-cli.js'
    at Function.Module._resolveFilename (module.js:547:15)
    at Function.Module._load (module.js:474:25)
    at Function.Module.runMain (module.js:693:10)
    at startup (bootstrap_node.js:188:16)
    at bootstrap_node.js:609:3
```
**第一反应**是网上直接搜索答案，有说需要重装`nodejs`的，但是重装后问题依旧，于是决定自己定位解决。

在重新审视了一下错误提示后，初步判定是文件缺失，然后根据在源文件夹中搜索`npm-cli.js`文件，发现其路径为：`/home/me/npm-6.1.0/bin/npm-cli.js`，既然文件存在，那应该是路径标识错误的问题了，于是打开`npm-6.1.0/bin/npm`，可以看到
```
NPM_CLI_JS="$basedir/node_modules/npm/bin/npm-cli.js"
```
这里表示读取的`npm-cli.js`的位置，基本上算是写死，并且位置与实际的位置有冲突，将这句注释，并在下面修改位置信息
```
NPM_CLI_JS="$basedir/npm-cli.js"
```
然后保存退出，再次运行`./npm -v`，得到正常版本信息。

于是将文件夹移动，并建立软连接
```
[me@localhost bin]$ sudo mv /home/me/npm-6.1.0/ /usr/local/npm
[me@localhost bin]$ sudo ln -s /usr/local/npm/bin/npm /usr/local/bin/npm
```
此时运行`npm -v`出现了错误：
```
module.js:549
    throw err;
    ^

Error: Cannot find module '/usr/local/bin/npm-cli.js'
    at Function.Module._resolveFilename (module.js:547:15)
    at Function.Module._load (module.js:474:25)
    at Function.Module.runMain (module.js:693:10)
    at startup (bootstrap_node.js:188:16)
    at bootstrap_node.js:609:3
```
继续分析错误，基本可以确定依旧是**文件位置定位错误**的问题，打开`/usr/local/npm/bin/npm`，可以找到：
```
basedir=`dirname "$0"`
```
它的位置指向的是**软连接文件**所在的路径，而非我们希望的**实际文件**所在的文件路径，于是只要将它修改成**实际文件**的绝对路径就好了：
```
basedir=$(dirname $(readlink -f $0))
```
修改完了之后，保存退出，再试下`npm -v`显示出正确的版本信息`6.1.0`，说明正确安装结束。
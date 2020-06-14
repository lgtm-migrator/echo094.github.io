---
layout: post
title:  "软件实用技巧"
date:   2018-06-21 00:00:00 +0000
categories: note
---

修改时间： 2019-08-04

一些比较实用的东西。



## Windows

### 删除网络连接信息

Windows系统会根据网关的mac地址识别网络，若是经常变更有线网络环境，会出现一堆多余的信息。清理方法如下：

1. 删除注册表路径 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged 下的无用条目
2. 删除注册表路径 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles 下的对应条目，条目名称与1中的ProfileGuid信息一致。



### Route配置

路由条目说明：

* 网络目标和网络掩码：该条目包括的IP地址范围。
* 网关：指向这些IP的数据被发送到的上级路由。
* 接口：该网关下的本机IP
* 跃点数：表示消耗，当一个IP符合多条路由时，系统选择跃点最小的。
* 在链路上：说明该网络目标被配置了多个网关。

一般情况下路由会自动配置好，但是在同时连接多个网络时，可能会出现网络的重叠。这时如果需要访问指定网络的资源，就需要手动修改路由表中的该IP条目了。



### IIS

**FTP**

端口范围需要在根目录下修改，在特定的站点打开防火墙设置的话，输入框为灰色。

IP需要填写防火墙的外网IP。

**Web Platform Installer**

无法修改配置、无法安装插件可能是权限问题，直接以管理员方式打开该安装程序即可解决问题。

参考：[Error: Saving Web Platform Settings in IIS](https://thycoticwiki.atlassian.net/wiki/spaces/KB/pages/1167387/Error+Saving+Web+Platform+Settings+in+IIS)



## 浏览器

### 禁用浏览器的WebRTC

 WebRTC用于建立浏览器之间点对点的连接，实现视频流和音频流或者其他任意数据的传输。一般不会用到这种服务，但是浏览器默认开启该服务时会有[这样](https://www.v2ex.com/t/98147)的风险，因此在必要时可以选择禁用浏览器的该功能。

对于Firefox，可以通过选项直接禁用：

1. 进入about:config设置页
2. 将media.peerconnection.enabled项置为false

对于Chrome可以安装以下插件并启用WebRTC相关策略：

* Privacy Badger 



## aria2

教程：[Aria2 + WebUI-Aria2](https://www.jianshu.com/p/870645c4b19f)

运行服务：

```shell
aria2c --enable-rpc --rpc-listen-all
```

运行WebUI：

```shell
node node-server.js
```

批量添加任务：

```
[node node-server.js] --out=[relative path]
```



## EndNote

配置项在注册表中，位置为`HKEY_CURRENT_USER\Software\ISI ResearchSoft\EndNote\Preferences`，同步路径可在此处修改。



## IDM

版本： `6.32 Build 2`

想要用IDM批量下载文件，又想根据URL保留文件的相对路径，然后发现这软件没这功能。于是想手动修改其保存路径，发现这软件并不简单。

IDM的下载记录保存在注册表中，路径为`Computer\HKEY_CURRENT_USER\Software\DownloadManager`。

下载任务计数为键值`maxID\maxID`，路径内以数字开头的文件夹就是一个个具体的任务了。

在任务目录内，其它的参数都可以直接读取，唯独对本地路径进行了简单的加密。保存本地路径的键值为`EncLNFSW`，加密方法如下：对于每一个字节，其高位不做处理，对于低位，使用15减去当前的值，所得到的值就是加密后的字节。于是解密也就很简单了。

然后就好办了，手动添加任务到注册表即可。只是这样做还是很麻烦，就改用aria2了。

下面附一个顺便发现的IDM数据备份工具：[[原创绿化] IDM备份管理器 | 自制绿色便携，最强大的IDM备份管理](https://www.52pojie.cn/thread-749647-1-1.html)



## Node

### npm与cnpm

**不建议使用cnpm源。**

有时从npm会下载失败，这时可以从镜像站下载，方式有两种。

第一种是在指令后面添加配置项，比如chromedriver：

```js
npm install chromedriver --chromedriver_cdnurl=http://cdn.npm.taobao.org/dist/chromedriver
```

第二种是在配置文件中添加配置项，比如electron：

```js
ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/
```

配置文件有全局以及用户两个，可通过该命令获取其位置：`npm config list`

### node-gyp

主页：[node-gyp](https://github.com/nodejs/node-gyp)

需要使用`python 2.7`，修改npm配置文件：

```
npm config set -g python /path/to/executable/python2.7
```

根据平台配置好相应的编译器。



## Visual Studio 2017

### 安装问题

如果是首次安装，除非是盗版系统，一般不会有什么问题。

如果是重新安装并且需要修改安装路径，就会发现有些路径无法修改，这时需要删除注册表中的相关内容。

**共享组件、工具和 SDK安装位置**

删除注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\Setup`下的`SharedInstallationPath`项

**WIN10SDK安装位置**

这个需要手动卸载。

删除注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows Kits\Installed Roots`下的`KitsRoot10`项

（非必须）删除注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Kits\Installed Roots`下的`KitsRoot10`项

### 调试问题

如果在调试时出现JS脚本错误的对话框，说明IE的版本太低了。


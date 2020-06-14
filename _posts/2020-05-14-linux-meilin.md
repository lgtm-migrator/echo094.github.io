---
layout: post
title:  "梅林KoolShare版配置说明"
date:   2020-05-14 00:00:00 +0000
categories: linux router meilin
---

修改时间： 2020-05-23

路由器： R7000



## CloudFlare DDNS

由于家宽只有公网ipv6，需要设置ipv6的DDNS，故根据帖子[[其他插件] 完美支持CloudFlare IPV6的DDNS插件](https://koolshare.cn/thread-175824-1-1.html)安装了插件。

在用的时候发现在自己的环境下存在两个问题，导致不能正确更新。

需修改的文件为：`/koolshare/scripts/cfddns_run.sh`。

第一个问题是方法`get_local_ipv6`返回的是v4的ip，因此改成下述代码，从路由器的状态信息中获取：

```sh
localipv6=`ip addr show ppp0 |grep inet6 |grep global |sed 's/\/.*//g' |awk '{print $2}'`
```

第二个问题是方法`get_info_ipv6`中CloudFlare返回的数据包有所改变，无法正确截取ip信息，故改为：

```sh
current_ipv6=`echo $cfddns_result |sed 's/.*"content":"//g' |sed 's/".*//g'`
```

修改后插件能够正确获取本地和域名的ip。

对于方法`get_info`，如果有需要也可做出相似的改动。

另外，如果该域名的ip记录不存在，仍然会返回`"success":true`信息，只是记录集为空：`"result:[]"`。由于这个问题不影响脚本的运行，就不修改了。



## Transmission

买了个硬盘，NAS还在等新款，只能暂时用路由器安装Transmission挂PT下载。

参照论坛的帖子：[梅林，安装transmission，开启外网远程，开启身份验证](https://koolshare.cn/thread-42506-1-1.html)，安装华硕下载大师的法子安装了Transmission，用是能用，但是好多种子连接不上服务器。

去论坛里查找了一下，发现是因为这个ipkg源下的openssl是0.9.8版的，不支持新的TLS协议。解决方法是使用entware，参考另一个帖子：[安装entware求助](https://koolshare.cn/thread-44816-1-1.html)。

步骤如下：

1. 给entware建立一个文件夹并挂载到`/tmp/opt`。

   在梅林系统下，路径`/opt`被链接到了`/tmp/opt`，因此改文件夹最终被链接到了`/opt`。

   ```bash
   cd /jffs
   mkdir /jffs/opt
   ln -nsf /jffs/opt /tmp/opt
   ```

   教程里该目录被创建在了`jffs`文件夹内，其实也可以创建在外接的USB设备内。

   由于之前安装过华硕下载大师，`/tmp/opt`被链接到了ipkg文件夹`/tmp/mnt/硬盘名称/asusware.arm`，在这里先给文件夹改个名字，以免开机时被自动挂载。

2. 安装opkg

   可以下载下面的脚本一键安装：

   ```bash
   wget http://qnapware.zyxmon.org/binaries-armv7/installer/entware_install_arm.sh
   sh ./entware_install_arm.sh
   ```

   但是脚本中的软件源下载比较缓慢，可以参照脚本中的内容手动下载并安装。

3. 自动挂载opt分区

   在`/jffs/scripts`位置建立文件`post-mount`，如果文件已经存在直接在里面添加内容：

   ```sh
   #!/bin/sh
   
   ln -nsf /jffs/opt /tmp/opt
   ```

   给文件添加执行权限，链接的位置工具实际路径调整。

在安装好entware后，就可以安装Transmission了，目前版本为2.84：

```bash
opkg update
opkg install transmission-cli transmission-web 
```

安装后会生成配置文件目录`/opt/etc/transmission`，可以手动修改`settings.json`。

安装后会添加启动脚本`S88transmission`到`/opt/etc/init.d`。

手动执行一下启动脚本运行Transmission，webUI的默认端口号是9091。


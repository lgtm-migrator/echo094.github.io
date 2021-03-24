---
layout: post
title:  "群晖系统中使用calibre"
date:   2021-03-24 00:00:00 +0000
tags: linux synology
---



按照说明操作没什么大问题，有些细节注意一下就行。只是这个软件在加载是是整本书一起加载，在书籍体积较大时很不方便。



系统： DSM 6.2.3-25426(DS920+)

资料：

1 [janeczku/calibre-web](https://github.com/janeczku/calibre-web/)



# 1 安装calibre数据库

使用`linuxserver/calibre`

* 映射目录`/config`，该目录用于保存图书。
* 不需要设置固定的端口映射，这个容器在建立好数据库后就不用去管他了
* **设置容器运行的用户和用户组**，添加两个环境变量即可，可以用现有的，也可以新建一个。

在创建后使用映射的`8080`端口进入设置，初始化数据库。



# 2 安装calibre-web

这里有两个web端，分别是**Technosoft2000**和**LinuxServer**的，这两个都可以，但设置有些许区别。

---

**使用Technosoft2000**

映射下述路径：

* `/books`：图书数据库目录，也可以用别的名称
* `/calibre-web/app`：web应用目录
* `/calibre-web/kindlegen`：`kindlegen`应用目录
* `/calibre-web/config`：配置文件目录

添加环境变量：

* `USE_CONFIG_DIR=true`：这样才能将配置文件保存到我们设置的外部目录
* `PGID`和`PUID`：用户和用户组，需要和上面calibre的一致，不然无法读取图书数据库。

**使用LinuxServer**

映射下述路径：

* `/books`：图书数据库目录，也可以用别的名称
* `/config`：配置文件目录

添加环境变量：

* `DOCKER_MODS=linuxserver/calibre-web:calibre`：添加ebook转换模块
* `PGID`和`PUID`：用户和用户组，如果两个都不设置的话也不会出问题，因为这两个容器的默认配置一致，但最好制定一下。

在web页面设置中添加转换模块的路径：

* `Path to Calibre E-Book Converter`：`/usr/bin/ebook-convert`
* `Path to Kepubify E-Book Converter`：`/usr/bin/kepubify`

---

在创建后进入映射的`8083`端口初始化，选择数据库所在目录。

默认用户名`admin`，密码`admin123`。

在设置中开启上传功能，其它的随意。

转换功能在书籍的编辑窗口中。


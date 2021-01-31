---
layout: post
title:  "群晖系统中使用tree命令"
date:   2021-01-31 00:00:00 +0000
tags: linux synology
---



虽然群晖基于linux，但是他很精简，啥都没有。在这里可以先安装ipkg包管理，再通过ipkg安装tree。

系统： DSM 6.2.3-25426(DS920+)

资料：

1 [群晖找不到apt yum解决办法 轻量级工具ipkg](https://blog.csdn.net/qq_37946291/article/details/108421382)



安装ipkg：

```bash
wget http://ipkg.nslu2-linux.org/feeds/optware/syno-i686/cross/unstable/syno-i686-bootstrap_1.2-7_i686.xsh
chmod +x syno-i686-bootstrap_1.2-7_i686.xsh
sudo sh syno-i686-bootstrap_1.2-7_i686.xsh
```

执行更新：

```bash
ipkg update
```

安装tree。



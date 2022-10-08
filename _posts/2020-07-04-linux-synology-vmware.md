---
layout: post
title:  "Vmware安装DSM6.2"
date:   2020-07-04 00:00:00 +0000
tags: linux synology
---



在给物理设备修改配置前，先用虚拟机试验一番。



更新日期：2021-04-01

系统： DSM 6.2-23739 (DS3617xs)



# 正文



在正式买硬件之前，先在虚拟机装个DSM熟悉一下系统环境，顺便把PT挂的种子转换过去。



## 1 安装黑群晖系统

照着下面这个帖子做就行了：

[群晖折腾二：初学者使用VMware 虚拟机安装群晖体验DSM6.2系统 （超详细版本）_NAS存储_什么值得买](https://post.smzdm.com/p/ar0v4orw/)



## 2 设置存储

系统已经安装在我们的第一个虚拟磁盘上了。为了安装套件，必须先建立一个存储空间。

1. 打开`Storage Manager`

2. 在`Storage Pool`标签页中创建一个存储池

   这里先用`Basic`试一下，硬盘选择上面划分的虚拟硬盘

3. 在`Volume`标签页中创建一个存储空间，这里使用`ext4`文件系统。

其实用RAID并不代表万无一失了，一般设备中的硬盘使用时间差不多，在RAID模式下损耗程度也十分接近。在坏了一块硬盘后，如果直接选择重建，高强度的读写操作很容易让其它硬盘也损坏。因此，在这种情况下，最佳的做法是先将所有的数据拷贝到一套新的硬盘中，然后用新的硬盘重建。但是这样的话还不如对于每一块硬盘用RAID1了。

我用NAS主要是为了存放影视资源，挂一下PT，以及内网文件共享，因此不需要反复写文件。一般是放满一个影片再加一个硬盘。在这种情况下，使用BASIC模式是很合理的选择。

文件系统选择`ext4`是考虑到它比较成熟，在系统出问题以后还能放到windows下恢复，相关内容见附加部分。



## 3 配置系统

1. 设置家目录

   在控制面板`User`标签页的高级中，下拉到最后`User Home`条目，点击启用。

2. 开启SSH

   在控制面板的`Terminal & SNMP`标签页中开启

3. 配置SMB

   添加共享文件夹就不用说了，对于PT用户，软链接是必不可少的，这样才能在不影响保种的前提条件下合理命名媒体资源。

   需要手动开启共享文件夹的软链接跟踪：

   在控制面板`File Services`标签页SMB的高级设置中开启软链接。



## 4 添加直连硬盘

装黑群一是方便目前的文件共享，二是在以后转白后能够直接把现有硬盘放上去。于是所有的资料都放在实体硬盘里，而套件等内容放在虚拟硬盘中。

在关闭状态下打开虚拟机的配置面板，选择添加硬盘，模式选择`SATA`，选择添加物理硬盘，并勾选整个硬盘。

添加以后开机，如果硬盘上没有群晖系统，必须走一遍添加存储空间的流程。如果已经安装了系统，系统版本不得高于当前系统版本（未测试，文档是这么说的）。



# 附加

黑群晖的安装和使用笔记到这里就结束了，下面是一些额外的备忘。



## 1 非群晖读取硬盘

如果哪天作死，系统出了问题，就要靠这个方法拯救数据了。

操作方法见官方文档：

[如何使用计算机恢复 Synology NAS 上的数据？](https://www.synology.com/zh-cn/knowledgebase/DSM/tutorial/Storage/How_can_I_recover_data_from_my_DiskStation_using_a_PC)

**在Linux环境下**

安装工具包：

```bash
sudo -i
apt-get update
apt-get install -y mdadm lvm2
```

查找RAID分区：

```bash
sudo mdadm -Asf && vgchange -ay
```

在完成上述的操作后，使用下列指令找到硬盘编号，我这里位于`/dev/md3`：

```bash
cat /proc/mdstat
```

然后使用挂载指令挂载即可。

```bash
# 只读
mount /dev/md3 /mnt/ -o ro
# 读写 保险起见最好别这么做
mount /dev/md3 /mnt/ -o rw
```

在这里只能看到存储空间中的内容，不能看到系统分区。

**在Windows环境下**

如果要看系统分区的文件，可以使用`DiskGenius`。软件中该硬盘表现为3个分区，第一个是就是系统分区（每个硬盘上都有1份系统），但文件分区显示为`Linux RAID`，即使分区格式为basic。



## 2 迁移硬盘

国行DS920+上市了，果断买了一台。拿到手先插一个空硬盘装系统，系统版本为（DSM 6.2.3-25426）。关机后把虚拟机挂载的磁盘按顺序放到新的NAS中（虚拟机的虚拟磁盘对应NAS的盘位1）。

开机以后硬盘能够正常读取，只需要修复一下系统分区，毕竟920的系统版本比黑群的高。

根据官方文档：[如何在多台 Synology NAS（DSM 6.0 和更新版本）之间进行迁移](https://www.synology.com/zh-cn/knowledgebase/DSM/tutorial/General_Setup/How_to_migrate_between_Synology_NAS_DSM_6_0_and_later)，使用**Hyper Backup**迁移时，发现qbittorrent等第三方套件都不能备份。这也不是大问题，可以把包含配置文件的目录备份，再在新设备中手动安装套件即可。



## 3 清理计划任务日志

参考下面的帖子: 
[How to purge task manager logs](https://www.reddit.com/r/synology/comments/osc8tb/how_to_purge_task_manager_logs/)

群晖计划任务的运行记录保存在数据库`sched_status.sqlite`的表`task_status`中，
数据库位于目录`/usr/syno/etc`，只需要清空对应`task_id`的记录即可。

```sql
select count(*) from task_status where task_id=1;
delete from task_status where task_id=1;
.quit
```

`task_id`可以使用下述命令查找：

```bash
/usr/syno/bin/synoschedtask --get
```



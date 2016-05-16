---
title: SDD+HHD Win10+Ubuntu 16.04双硬盘双系统U盘安装
date: 2016-05-16 09:18:08
tags:
   - Ubuntu
   - Windows

---

# 主要流程

1.  对已安装Windows进行磁盘压缩
2.  下载Ubuntu 16.04镜像软件，制作U盘启动盘使用ultraISO
3.  安装Ubuntu系统
4.  用EasyBCD 创建启动系统启动引导

# Windows磁盘压缩
　　这里默认安装了Windows10系统，由于我的Windows10安装在SSD，那么Ubuntu引导/boot也需要安装在SSD比较好。当然你可以选择安装在HHD,不过后面引导的部分可能很麻烦。右键：此电脑-管理-磁盘管理， 对SDD右键压缩卷，压缩出几百M进行Ubuntu的/boot安装，在HHD压缩出50G进行Ubuntu其它的挂载，可以根据你的情况划分多点。

<!--more-->

<img width = "80%" src= "http://7xtwmz.com1.z0.glb.clouddn.com/win-1.png">

# 制作Ubuntu的U盘启动
到官网免费下载[Ubuntu16.04](http://www.ubuntu.com/download/desktop),使用UltraISO进行制作U盘启动。

<img width = "80%" src= "http://7xtwmz.com1.z0.glb.clouddn.com/ubuntu-1.png">

<img width = "80%" src= "http://7xtwmz.com1.z0.glb.clouddn.com/ubuntu-2.png">

<img width = "80%" src= "http://7xtwmz.com1.z0.glb.clouddn.com/ubuntu-3.png">

# 安装Ubuntu系统
　　重启按F2（不同机子不同）到BIOS设置Boot启动为U盘，按照安装步骤进行，分区选“其他选项”,进行下面的分区设置：

1. 逻辑分区，500M，起始，Ext4日志文件系统，/boot (引导分区挂载到你的SSD压缩出来的空间)
2. 逻辑分区，5000M，起始，交换空间，无挂载点；（交换分区swap，一般不大于物理内存）
3. 逻辑分区，15000M，起始，Ext4日志文件系统，/；（系统分区)
4. 逻辑分区，剩余空间数，起始，Ext4日志文件系统，/home；（home分区存放个人文档）

# 用EasyBCD创建启动系统启动引导
<img width = "80%" src= "http://7xtwmz.com1.z0.glb.clouddn.com/win-u.png">
重启之后就可以看到Windows10和Ubuntu的启动选择项了。


# 参考

[Win7 U盘安装Ubuntu16.04 双系统详细教程](http://blog.csdn.net/coderjyf/article/details/51241919)

[SDD+HDD+Win7+Ubuntu12.04双硬盘双系统安装流程](http://jingpin.jikexueyuan.com/article/36416.html)

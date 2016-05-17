---
title: BIOS下U盘无法启动
date: 2016-05-16 13:22:39
tags:
    - Windows

---
BIOS设置中，无法启动U盘，多出现在WIN8的系统和华硕的品牌机上。

选择“UEFI：u盘”启动项的时候，无法进入U启动, 一般是由于Launch CSM系统兼容支持模块没有启用。
LaunchCSM选项是灰色的，意思就是说我们选择不了它那个选项，也就修改不了它的状态。这是由于BIOS  设置了 Secure  Boot  Control (安全启动控制)。

我们可以在在主菜单Secure/Security目录下，选择Secure  Boot  Control 把它设置为Disabled。

然后我们在BOOT 菜单下的Launch CSM设置为Enabled（启用）即可。

重启电脑再次进入BIOS设置，我们便可看见，在启动选项里面，出现多了几个启动选项。

<!--more-->
有时选择了“UEFI：u盘”做为第一启动的时候，电脑还是无法进入PE。这是因为选择了接口方式不一样的启动方式。

　　这时有两个u盘启动，一个是带着UEFI:格式的，一个是不带任何格式的。UEFI:新型UEFI，全称“统一的可扩展固件接口”(Unified Extensible Firmware Interface)，是一种详细描述类型接口的标准。这种接口用于操作系统自动从预启动的操作环境，加载到一种操作系统上。如果我们选择了UEFI这种接口方式是进不了PE的，所以我们选择另一个不指定接口方式的u盘启动作为第一启动即可。

　　如果进入PE以后，发现没有硬盘，是因为硬盘传输模式选择错误。硬盘传输模式分两种设置：现在新的笔记本都是用SATA接口硬盘，但硬盘传输方式有两种设置：一种是SATA模式（AHCI、增强模式）；另一种是IDE（ATA、兼容模式）。由于我们的u盘没有集成SATA控制器驱动，所以安装过程中会找不到硬盘。而如果我们在BIOS中把硬盘设置为兼容模式（IDE模式）就可以安装了。进到BIOS后，选择Main下的IDE Configuration Menu，在Onboard IDE Operate Mode下面可以选择两种IDE操作模式：兼容模式和增强模式(Compatible Mode和Enhanced Mode)。其中兼容模式Compatible Mode。有些电脑bios设置可能不一样，会在Advanced菜单下的SATA Configuration（硬盘配置）选择菜单中我们选择IDE兼容模式，按F10保存即可。

如图

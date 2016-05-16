---
title: windows10下CPU磁盘运行总是100%优化方法
date: 2016-05-12 09:05:51
tags:
   - Windows
---
# 问题
Windows10新系统运行会出现问题：
    - 磁盘经常100%运行；
    - cpu经常100%运行；
    - 内存的使用量占的更大；
下面介绍一下怎么优化：

---

# 磁盘

## 关闭虚拟内存：
注：前提是电脑的内存足够大，建议电脑内存有8G及其以上者使用，这种方法的优化力度最大；
 1. 打开 我的电脑->右键属性->高级系统设置->系统属性->高级->性能 设置->性能选项->虚拟内存 -> 更改
 2. 取消 “自动管理所有驱动器的分页文件大小”
 3. 选择 “无分页文件”
 4. 然后依次点击 设置->确定-> 应用->确定->应用->确定

<!--more-->

## 关闭家庭组：
 1. 使用快捷键 Win + R 调出运行窗口;
 2. 输入 “services.msc”->确定 调出服务窗口;
 3. 通过  "HomeGroup Listener"->右键属性->服务状态 停止->启动类型 禁用->应用->确定 来停止该服务；
 4. 通过  "HomeGroup Provider"->右键属性->服务状态 停止->启动类型 禁用->应用->确定 来停止该服务；  


## 关闭windows默认的杀毒软件 Windows Defender：
 1. 使用快捷键 Win + R 调出运行窗口;
 2. 输入 “gpedit.msc”->确定 调出本地组策略编辑器窗口;
 3. 依次选择 本地计算机策略->计算机配置->管理模板->Windows组件->Windows Defender；
 4. 双击右边的 "关闭Windows defender"；
 5. 选择 “已启用”->“应用”->“确定”


# CPU

## 关闭windows默认的杀毒软件 Windows Defender：
Windows自带的杀毒软件会不定时进行磁盘扫描，查杀病毒，关闭杀毒软件应该好理解，具体操作同上；

## 关闭“Diagnostics Tracking Service”服务；
1. 同上“磁盘->关闭家庭组”的操作打开 服务窗口；
2. 关闭 “Diagnostics Tracking Service”服务；


# 内存

## 关闭使用内存多的程序：
1. 通过快捷键 “Ctrl+Shift+Esc”打开任务管理器；
2. 双击“内存”列进行倒叙排序；
3. 选择不用的应用 右键“结束任务”

## 添加更多的内存条：
1. 添加内存条需要电脑主板的支持.
2. 对于32位的操作系统，系统支持的最大的内存是4G，64位的系统最大支持2的34次方G的内存.
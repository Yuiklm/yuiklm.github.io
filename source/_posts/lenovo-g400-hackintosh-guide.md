---
layout: post
title: 「 联想 G400 笔记本 」 Hackintosh 攻略
draft: false
date: 2018-06-21 23:21:27
categories:
tags: hackintosh,g400
permalink:
description:
cover_img:
toc-disable:
comments:
---
#### 🛠 准备工作 

* 笔记本： 联想 G400
* U 盘：SanDisk 酷铄 CZ73
* 内存条：金士顿 DDR3L 1600Mhz  4G x1
* 无线网卡&蓝牙：博通 BCM94352HMB

#### 🔧 BIOS 移除白名单 

在拿到 **博通BCM 94352HMB** 和  **内存条** 当天，就都装了上去
 <!-- more -->
 
 <img src="http://wx2.sinaimg.cn/mw690/699458aegy1fsj7cdyv76j21100o0hdu.jpg" width="500" />

提示 *Unauthorized network card is plugged....*  （当场 *白仁面* 去），意识到是联想坑爹的白名单限制无线网卡的型号，于是默默的打开 **google** ... 

于是，很快搜到 [Y400/Y500 史上最详细 英特尔7260AC+4.0蓝牙 究极白名单完美蓝牙改造方案](http://bbs.zol.com.cn/nbbbs/d160_143691.html) 、[LENOVO G400S 刷bios去除白名单](http://bbs.pcbeta.com/viewthread-1593118-1-1.html) ，大概阅读了下，发现并不复杂，几个步骤即可。

鉴于这次的 G400 是我家宝宝的，笔记本交给我之前下了死命令不能弄丢里面的数据，所以在刷 Bios 的时候还是挺怕刷炸主板的，还好过程一切顺利。

##### 准备工具：

 <img src="http://wx1.sinaimg.cn/mw690/699458aegy1fskh0pa47bj21d00ugn2w.jpg" width="550" />
 
*  78cn25ww ：BIOS 最新版
*  启动U盘制作 ： 制作 U 盘为 DOS 启动盘
*  insyde EzH2O中文版 ： 编辑 BIOS ，移除白名单模块
*  tlzh：备份、写入 BIOS

##### 简单步骤：

简单讲，需要把 BIOS 的白名单模块去除，这样开机的时候就不会检测无线网卡，可以顺利进入系统；把白名单去除需要把 BIOS 备份出来，然后用 **insyde EzH2O** 把模块去除，然后将修改后的 BIOS 写入，备份和写入在 DOS 环境工作。

 <img src="http://wx4.sinaimg.cn/mw690/699458aegy1fskhgwvmkdj21kw16ou0x.jpg" width="500" />

值得一提的是，在使用 **insyde EzH2O** 移除白名单模块的时候，因为上述文章机型并不是 G400 ，所以移除的时候不太确定文中的 `GUID "11D378C2-B472-412F-AD87-1BE4CD8B33A6" ` 是不是对应 G400 的无线网卡白名单模块，但是网上搜不到 G400 的，加上都是联想的机器，想着应该都是一样的，就直接删除这个模块，这个过程忘记截图拍照... 最终成功开机，意味着白名单模块成功移除，win 系统下 wifi 📶 正常。

#### ⚙️ Mac 系统 U 盘制作、分区

下载带 **Clover** 的 [macOS High Sierra 原版镜像](http://bbs.pcbeta.com/viewthread-1787600-1-1.html)  ，该镜像自带适合各个机型的 Clover 的配置文件以及涵盖大部分网卡、显卡、万能声卡驱动

如果已经有 macOS 平台，在磁盘工具中利用恢复功能将镜像恢复到 U 盘即可，恢复成功后 u 盘会自动分成两个分区，一个存放  macOS ，另一个 EFI 分区存放 Clover 用于启动引导。

 <img src="http://wx2.sinaimg.cn/mw690/699458aegy1fsuurx9ewsj21kw11n7bm.jpg" width="500" />

如果你没有 macOS 平台，那么需要 **Tranmacs** 来将 dmg 文件写入 U 盘，同样写入完成会生成两个分区。

【这里应有图片】

EFI 分区默认隐藏，在 mac 中借用 Clover Configurator 可以打开隐藏了的 EFI 分区，在 Clover 目录中，选择适合自己电脑配置的 config 文件，只要选择正确，一般都能正常走完安装流程。

 <img src="http://wx3.sinaimg.cn/mw690/699458aegy1fsuus3jntlj21kw0xuaqr.jpg" width="500" />

#### 🗡 安装 macOS

安装之前先确保 EFI 分区 >= 200M ，且已为 macOS 准备好一个分区（此过程建议在 PE 中进行），并在 Bios 中关闭 Secure Boot 。

用写好镜像和 Clover 的 U 盘启动电脑，选择 macOS install ，耐心等待，待成功进入安装界面后将准备要装 macOS 的分区抹掉，选择 **Mac OS 扩展** 格式，抹掉成功后选择安装在该分区即可，安装过程没有什么需要注意的，无脑下一步就可以了，重新继续安装，配置账号信息等，直到安装完毕进入桌面。

过程中，除了第一次 Clover 引导选择页面需要选择 masOS install（即写入到 u 盘的 macOS 安装镜像），第二次开始到以后都选择划分给 macOS 的分区进入系统。 

#### ⚽️ 完善驱动、DSDT、SSDT

此时仍然以准备的 U 盘来启动系统，一般 Clover 的配置文件选择正确，加上不少自带的驱动，所以第一次进入系统显卡、声卡（万能声卡）、网卡（有线无线）、蓝牙都能成功驱动，剩余 *电池电量、屏幕亮度控制、变频* 等利用 DSDT、SSDT 解决。

>  [MaciASL](https://github.com/RehabMan/OS-X-MaciASL-patchmatic) 是一个苹果原生的，可以用来 修改、编译 DSDT、SSDT等ACPI文件的软件  
>  [Rehabman的综合补丁源](https://github.com/RehabMan/Laptop-DSDT-Patch) Rehabman大神的 DSDT 补丁源，如果你的机型在补丁列表里面，那么恭喜，直接打上该补丁即可

[[授权翻译] 使用补丁修改DSDT/SSDT [DSDT/SSDT综合教程]](http://bbs.pcbeta.com/viewthread-1571455-1-1.html) 配合 [补丁源](https://github.com/RehabMan/Laptop-DSDT-Patch) 食用，打上 电池电量、屏幕亮度控制 等几个必要补丁，复制到 clover 中，重启即可生效。

#### 👾 收尾

最后把完整的 Clover 复制到硬盘上的 EFI 分区，至此圆满安装成功，可脱离 U 盘使用。

在简单甩几个链接之后，攻略也就这样结束了，仅此作为记录 **黑苹果** 的折腾思路参考，安装过程中或多或少（根据到目前为止安装超过5台笔记本的、大学3年黑苹果使用经验）会有各种问题，推荐在*[远景](bbs.pcbeta.com)*中搜一下一般都能解决。祝好运～ 

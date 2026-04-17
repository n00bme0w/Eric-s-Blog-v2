---
title: "使用fastboot安装android16 qpr1 userdebug gis系统踩坑记录"
date: 2026-04-17
draft: false
description: "由于user版本的android16机制过于复杂，magisk很难绕过，我决定直接刷机成userdebug版本..."
tags: []
---
# 背景

我需要替换安卓手机的art组件来进行性能测试，以支持art开发。但是由于我的手机是user版本，即使通过magisk替换art组件，apexd仍然由于版本验证和哈希验证等原因拒绝使用我的art。尝试了各种方案，包括修改magisk脚本，更改art组件版本号等方式后仍未有效果，因此我转向重刷机成userdebug版本。

# 过程

## 准备镜像

首先是官方的android16根本不提供userdebug版本，并且由于android16后的aosp不提供设备目录树，且pixel10之后的设备驱动不开放，所以尝试自己编译也是困难的。
最后我从社区找到了编译完成的[android16 userdebug gsi](https://sourceforge.net/projects/andyyan-gsi/files/aosp-pure-userdebug/)镜像，选择最接近我原本版本的下载到服务器上。

除了要刷进去的版本的镜像，还必须准备手机原厂镜像。从[官网](https://developers.google.cn/android/images?hl=zh-cn)下载即可

解压userdebug img.gz文件，得到img文件。

解压原厂镜像，得到vbmeta.img

## 刷入镜像

这里比较容易踩坑。

首先澄清一点

``` bash
adb reboot bootloader
```
可以进入手机的fastboot模式

而
``` bash
adb reboot fastboot
```

可以进入手机的fastbootd模式

这两个模式不应该混用。

首先
``` bash
adb reboot bootloader
```
进入fastboot模式

然后
``` bash
fastboot --disable-verification flash vbmeta vbmeta.img
```
刷入原厂vbmeta.img

然后

``` bash
fastboot reboot fastboot
```

进入用户空间

然后删除原system分区

``` bash
fastboot erase system
```

再删除product分区(刷入gsi时需要，刷入原厂系统不需要)
``` bash
fastboot delete-logical-partition product_b
```

这里有可能显示找不到delete-logical-partition命令，那就说明fastboot版本太老

鉴于我没有sudo权限，不能使用apt升级，我直接在aosp源码中make fastboot。然后使用编译的新版fastboot工具

再刷入系统镜像

``` bash
fastboot flash system system.img
```

这个时候出现错误，发现无法刷入。

经检查发现是社区编译的镜像文件不是simg(sparse image)格式，同样由于没有sudo安装比较复杂，我直接通过aosp编译获得img2simg工具

``` bash
make img2simg-host
```

使用该工具获得simg镜像

然后重新尝试指令刷入

刷入完毕后通过fastboot进入recovery

``` bash
fastboot reboot recovery
```

必须在recovery界面手动格式化data分区

根据[知乎上的教程](https://zhuanlan.zhihu.com/p/705143850)的说法，在新版安卓使用fastboot -w会破坏super分区，导致刷机失败

重启后刷机成功。


## 提示

如果在刷入system镜像时卡住，长时间无反应，可能是忘记重启进入fastbootd模式。在新版安卓中，必须在fastbootd模式下刷写system等动态分区。



---
title: "关于pixel手机网络与时钟同步问题"
date: 2026-04-13
draft: false
description: "调试心得"
tags: []
---


之前在服务器的pixel手机进行profiling遇到了一些问题，主要体现为WIFI图标显示小叹号，知乎，小红书，抖音等app无法正确加载内容，显示联网错误，且登录显示时钟不正确。

故而从网上寻求答案，通过[他人博客](https://blog.ugoearn.com/pixel_fix_network-_time/)获取解法


在飞行模式执行
``` bash
adb shell settings delete global captive_portal_https_url
adb shell settings delete global captive_portal_http_url
adb shell settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204
adb shell settings put global captive_portal_https_url https://connect.rom.miui.com/generate_204
```
修复WIFI叹号问题


执行
```bash
adb shell "settings put global ntp_server pool.ntp.org"
```

然后重启
修复时钟不同步问题

结果是知乎和小红书可以登录，抖音可以正确加载视频，但可能由于刷过过多脚本，依旧无法登录


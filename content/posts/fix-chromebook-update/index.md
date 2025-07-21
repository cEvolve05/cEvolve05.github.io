+++
date = "2024-05-24T19:59:00+08:00"
# lastmod = "2025-07-21T21:51:01+08:00"
draft = false
title = "修改 HWID 让 Chromebook 工程机获取自动更新"
# summary = ""
+++

最近买了一台 Chromebook，卖家说是工程机，无法自动更新，在网上找到解决方法，记录如下。

我这台工程机的主要问题是设备的 HWID 不正确，导致 ChromeOS 无法识别设备，所以只要修改 HWID 即可。

> **前提条件**
> 
> 1. 开启开发者模式（会清除数据）
> 2. 解除固件写入保护
> 
> [ChromeOS 固件布局介绍](https://mrchromebox.tech/#firmware)  
> [ChromeOS 开发者模式 介绍和开启方法](https://mrchromebox.tech/#devmode)  
> [ChromeOS 固件写入保护 介绍和关闭方法](https://wiki.mrchromebox.tech/Firmware_Write_Protect#Hardware_Write_Protection)

使用 MrChromebox.tech 提供的脚本修改 HWID，[介绍和使用方法](https://mrchromebox.tech/#fwscript)。

如果你找不到相同机型的 HWID，打开 [recovery.conf](https://dl.google.com/dl/edgedl/chromeos/recovery/recovery.conf)，查找自己的机型，`hwidmatch` 字段是机型 HWID 的正则表达式。

比如我的机型是 `IdeaPad Flex 5i Chromebook (13", 6)`，`hwidmatch` 对应为 `^LILLIPUP-MQUZ.*`，那么我就写成 `LILLIPUP-MQUZ 000-000-000-000`，后面的一串0是我在其他地方搜索到的可能的格式。你可以尝试搜索其他人分享的 HWID，打开 `chrome://system` 搜索 hwid 也可以找到当前的 HWID。

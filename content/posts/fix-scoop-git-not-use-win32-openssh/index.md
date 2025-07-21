+++
date = "2024-10-05T17:16:00+08:00"
# lastmod = "2025-07-21T21:38:16+08:00"
draft = false
title = "修复 Scoop 安装的 Git 不使用外部 OpenSSH"
# summary = ""
+++

## 环境&背景

- (Scoop) Git (<https://github.com/git-for-windows/git>)
- (Scoop) OpenSSH (<https://github.com/PowerShell/Win32-OpenSSH>)
- ssh 密钥已用密码加密
- ssh 密钥已添加到 ssh-agent (Win32-OpenSSH)
- Windows 自带的 OpenSSH 和另外安装的 Win32-OpenSSH 均无效
- 官网的安装器选择“使用外部openssh”后，签名提交时不需要输入密码，但是用 Scoop 安装的版本每次都需要密码

一开始以为 git 无法正确调用 ssh-agent，然后整个下午的搜索方向全歪了（）

## 原因

在安装 Git 的时候，自带一个 OpenSSH，同时 Scoop 使用的 portable 版本安装时不询问是否使用外部 OpenSSH，导致 Git 会尝试使用自带的 OpenSSH。

当使用官方安装器的时候会询问是否使用外部 OpenSSH，如果选择“是”的话，安装器会把自带的 OpenSSH 直接删掉，我当时查到这一点的时候都惊呆了（

## 解决方案

参考：<https://github.com/git-for-windows/git/issues/4960>

打开到 `<scoopDir>\apps\git\current\usr\bin`，搜索所有和 openssh 有关的可执行文件，改名或直接删除；或者修改 `gpg.ssh.program` (git config) 指向外部的 ssh-keygen 即可。

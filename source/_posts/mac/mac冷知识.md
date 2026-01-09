---
title: mac冷知识
date: 2025-12-24 14:25:07
categories: mac
tag: 冷知识
---

修复"打开应用程序时，提示程序损坏，扔进废纸篓！"
```shell
sudo xattr -r -d com.apple.quarantine /Applications/BongoCat.app
```
小米手机连接无线adb
```shell
 adb pair 172.18.6.86:43703
Enter pairing code: 250904
Successfully paired to 172.18.6.86:43703 [guid=adb-2e1ee8c4-rV5w7C]
```
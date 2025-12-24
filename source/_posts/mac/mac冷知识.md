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

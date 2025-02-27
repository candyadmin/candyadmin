---
title: logstach安装和导入数据
date: 2025-02-27 14:09:00
tags:
---

使用[logstash.conf](../../resource/logstash.conf)

运行logstash
```shell
wget https://artifacts.elastic.co/downloads/logstash/logstash-oss-7.1.0.tar.gz
tar -zxvf logstash-oss-7.1.0.tar.gz
cd logstash-oss-7.1.0
bin/logstash -f logstash.conf
```

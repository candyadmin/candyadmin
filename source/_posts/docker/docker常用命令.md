---
title: docker常用命令
date: 2025-02-20 15:26:30
categories: docker
tag: 命令
---

新建数据库：

```bash
docker run -d --restart=always  --name my-mysql -e MYSQL_ROOT_PASSWORD='password' -p 3306:3306 -e TZ=Asia/Shanghai  mysql:8.0.39
```
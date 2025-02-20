---
title: mysql-数据备份脚本
date: 2025-02-19 17:31:43
categories: mysql
tag: 备份
---


备份：

```bash
docker exec mysql-container mysqldump -u root -p'password' --all-databases > /path/to/backup/all_databases_backup.sql
```
还原：

```bash
docker exec -i my-mysql mysql  -u root -p'password'    < /opt/mysql_back/mydatabase_backup.sql  
```


新建数据库：

```bash
docker run -d --restart=always  --name my-mysql -e MYSQL_ROOT_PASSWORD='password' -p 3306:3306 -e TZ=Asia/Shanghai  mysql:8.0.39
```



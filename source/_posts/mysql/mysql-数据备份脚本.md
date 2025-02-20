---
title: mysql-数据备份脚本
date: 2025-02-19 17:31:43
categories: mysql
tag: 备份
---

如果你想备份 MySQL 容器中的 **所有数据库**，可以使用 `--all-databases` 参数。

命令如下：

```bash
docker exec mysql-container mysqldump -u root -p'password' --all-databases > /path/to/backup/all_databases_backup.sql
```

### 解释：
- `-u root`：指定 MySQL 用户名为 `root`。
- `-p'password'`：指定密码为 `password`（注意不要在 `-p` 和密码之间加空格，或者使用引号包裹密码）。
- `--all-databases`：备份所有数据库。
- `> /path/to/backup/all_databases_backup.sql`：将备份文件保存到宿主机的指定路径。

### 注意：
1. 确保在运行命令时提供正确的密码，避免出现提示要求输入密码的情况。
2. 如果备份过程中需要排除某些特定数据库，可以使用 `--ignore-database` 参数来指定不备份的数据库。

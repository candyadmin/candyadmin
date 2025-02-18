---
title: docker修改应用为自动启动
date: 2024-07-23 14:39:38
categories: linux
tag: docker
---
是的，Docker 提供了一种简单的方法来配置容器在系统启动时自动启动，这就是使用 `docker update` 命令配置容器的重启策略。

以下是如何使用 `docker update` 命令来配置现有的容器在系统启动时自动启动：

```bash
docker update --restart unless-stopped <container_name_or_id>
```

例如，如果你的容器名为 `mysql-container`，你可以这样配置：

```bash
docker update --restart unless-stopped mysql-container
```

这样配置后，当系统重启时，Docker 会根据 `unless-stopped` 策略自动重启该容器。以下是一些常用的重启策略：

- `no`: 容器不会自动重启（默认值）。
- `on-failure`: 仅当容器退出状态非零时才重启。
- `always`: 无论退出状态如何，总是重启容器。
- `unless-stopped`: 容器除非被手动停止，否则总是重启。

### 示例

以下是一个完整的示例，展示如何启动一个新的 MySQL 容器并配置其在系统重启时自动启动：

1. 启动一个 MySQL 容器：

    ```bash
    docker run -d \
        --name mysql-container \
        -e MYSQL_ROOT_PASSWORD=my-secret-pw \
        -v /my/own/datadir:/var/lib/mysql \
        mysql:latest
    ```

2. 配置容器的重启策略：

    ```bash
    docker update --restart unless-stopped mysql-container
    ```

### 检查容器重启策略

你可以使用以下命令来检查容器的重启策略：

```bash
docker inspect -f '{{ .HostConfig.RestartPolicy.Name }}' mysql-container
```

这将输出容器的当前重启策略。

### 使用 Docker Compose

如果你使用 Docker Compose，可以在 `docker-compose.yml` 文件中配置重启策略：

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:latest
    container_name: mysql-container
    environment:
      MYSQL_ROOT_PASSWORD: my-secret-pw
    volumes:
      - /my/own/datadir:/var/lib/mysql
    restart: unless-stopped
```

然后，使用以下命令启动容器：

```bash
docker-compose up -d
```

### 总结

通过上述方法，你可以轻松地配置 Docker 容器在系统重启时自动启动，无论是通过 `docker run`、`docker update` 还是 `docker-compose`。
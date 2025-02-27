---
title: 在Docker容器中运行Elasticsearch,Kibana和Cerebro
date: 2025-02-27 10:42:36
categories: elasticsearch
tag: docker部署
---

docker-compose.yml如下
```shell
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3
    container_name: cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://elasticsearch:9200
    networks:
      - es7net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.1.0
    container_name: kibana7
    environment:
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_01
    environment:
      - cluster.name=geektime
      - node.name=es7_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - es7net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_02
    environment:
      - cluster.name=geektime
      - node.name=es7_02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data2:/usr/share/elasticsearch/data
    networks:
      - es7net

volumes:
  es7data1:
    driver: local
  es7data2:
    driver: local

networks:
  es7net:
    driver: bridge
```
这个 `docker-compose.yml` 文件用于配置并部署一个包含 **Elasticsearch**、**Kibana** 和 **Cerebro** 的集群环境。

#### 1. **cerebro**
- **image**: 使用 `lmenezes/cerebro:0.8.3` 这个镜像，Cerebro 是一个开源的 Elasticsearch 管理工具。
- **container_name**: 将容器命名为 `cerebro`。
- **ports**: 将主机的 9000 端口映射到容器内的 9000 端口，允许通过浏览器访问 Cerebro 界面。
- **command**: 启动时设置参数，`-Dhosts.0.host=http://elasticsearch:9200` 指定 Cerebro 连接的 Elasticsearch 地址为 `elasticsearch:9200`。
- **networks**: 配置 `cerebro` 服务连接到 `es7net` 网络。

#### 2. **kibana**
- **image**: 使用 `docker.elastic.co/kibana/kibana:7.1.0` 镜像，这是 Elasticsearch 官方的 Kibana 7.1.0 版本。
- **container_name**: 容器命名为 `kibana7`。
- **environment**: 设置了一些 Kibana 配置项：
    - `I18N_LOCALE=zh-CN`：设置 Kibana 界面语言为中文。
    - `XPACK_GRAPH_ENABLED=true`、`TIMELION_ENABLED=true`：启用一些高级功能。
    - `XPACK_MONITORING_COLLECTION_ENABLED="true"`：启用监控功能。
- **ports**: 映射主机的 5601 端口到容器的 5601 端口，用于通过浏览器访问 Kibana 界面。
- **networks**: `kibana` 服务连接到 `es7net` 网络。

#### 3. **elasticsearch**
- **image**: 使用 `docker.elastic.co/elasticsearch/elasticsearch:7.1.0` 镜像，这是 Elasticsearch 7.1.0 的官方版本。
- **container_name**: 容器命名为 `es7_01`。
- **environment**: 设置 Elasticsearch 的环境变量：
    - `cluster.name=geektime`：集群名称为 `geektime`。
    - `node.name=es7_01`：节点名称为 `es7_01`。
    - `bootstrap.memory_lock=true`：启用内存锁定，防止虚拟内存交换。
    - `ES_JAVA_OPTS=-Xms512m -Xmx512m`：设置 JVM 的初始内存和最大内存为 512MB。
    - `discovery.seed_hosts=es7_01,es7_02`：指定集群中的其他节点用于发现集群成员。
    - `cluster.initial_master_nodes=es7_01,es7_02`：设置初始的主节点，用于集群初始化。
- **ulimits**: 配置容器的内存锁定，`soft` 和 `hard` 都设置为 -1，表示没有限制。
- **volumes**: 使用 `es7data1` 持久化数据。
- **ports**: 映射主机的 9200 端口到容器的 9200 端口，允许通过 HTTP 访问 Elasticsearch。
- **networks**: `elasticsearch` 服务连接到 `es7net` 网络。

#### 4. **elasticsearch2**
- **image**: 使用相同的 `docker.elastic.co/elasticsearch/elasticsearch:7.1.0` 镜像。
- **container_name**: 容器命名为 `es7_02`。
- **environment**: 与 `elasticsearch` 配置类似，唯一的区别是 `node.name` 设置为 `es7_02`，这是另一个节点。
- **ulimits**: 配置内存锁定，和 `es7_01` 节点一致。
- **volumes**: 使用 `es7data2` 持久化数据。
- **networks**: `elasticsearch2` 服务连接到 `es7net` 网络。

### `volumes`
- **es7data1** 和 **es7data2**：定义了两个持久化数据卷，分别用于 Elasticsearch 节点 `es7_01` 和 `es7_02` 的数据存储，使用 `local` 驱动。

### `networks`
- **es7net**: 定义了一个名为 `es7net` 的网络，所有的服务都将连接到这个网络上。网络使用 `bridge` 驱动，适用于 Docker 容器间的通信。

### 总结
这个 `docker-compose.yml` 文件定义了一个完整的 Elasticsearch 集群（包含两个节点），同时部署了 Kibana 和 Cerebro 来进行集群管理和可视化。通过将这些服务连接到一个共享的网络 `es7net`，它们能够相互通信。数据卷 `es7data1` 和 `es7data2` 保证了 Elasticsearch 的数据持久性，即使容器停止或重启，数据也不会丢失。
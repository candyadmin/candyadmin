---
title: grafana+prometheus+springboot监控平台
date: 2025-12-03 11:10:13
categories: spring-boot
tag: grafana,prometheus,springboot
---
好的，我现在提供一个完整的 Spring Boot 项目结构，您可以直接创建文件并运行。

请确保您的项目使用 **Java 17+** 和 **Spring Boot 3.x**，或使用兼容的旧版本。

### 1\. 项目依赖文件：`pom.xml` (Maven)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version> <relativePath/> 
    </parent>
    <groupId>com.example</groupId>
    <artifactId>grafana-metric-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>grafana-metric-demo</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
        
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### 2\. 配置文件：`src/main/resources/application.properties`

```properties
# 启用所有 Actuator 端点（包括 /metrics, /prometheus 等）
management.endpoints.web.exposure.include=*

# 确保 Prometheus 端点是启用的
management.endpoint.prometheus.enabled=true

# 设置应用端口
server.port=8080
```

### 3\. 指标服务：`src/main/java/com/example/demo/AccountMetricService.java`

(假设您的包名为 `com.example.demo`)

```java
package com.example.demo;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Tags;
import org.springframework.stereotype.Service;

@Service
public class AccountMetricService {

    private final MeterRegistry meterRegistry;

    public AccountMetricService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    /**
     * 对特定错误进行埋点计数：account.error.count
     * * @param type 错误的分类 (例如："validation", "database")
     * @param errorMessage 具体的错误信息 (例如："UserNotFound")
     */
    public void recordError(String type, String errorMessage) {
        
        // 使用 Counter.builder() 来创建或获取一个已注册的 Counter
        Counter.builder("account.error.count")
               .description("Counts errors in the account service by type and message")
               // 动态添加标签 (Tags)
               .tags(
                   "tag_type", type == null ? "unknown" : type,
                   "tag_error", errorMessage
               )
               .register(meterRegistry)
               .increment(); // 执行计数加 1
        
        System.out.println("--- 埋点计数: type=" + type + ", error=" + errorMessage + " ---");
    }
}
```

### 4\. 测试接口：`src/main/java/com/example/demo/MetricTestController.java`

我们创建一个 Controller 来模拟业务逻辑，方便通过浏览器或 cURL 来触发计数器。

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MetricTestController {

    private final AccountMetricService metricService;

    public MetricTestController(AccountMetricService metricService) {
        this.metricService = metricService;
    }

    /**
     * 访问此接口来触发错误计数。
     * 示例 URL: http://localhost:8080/record-error?type=database&msg=ConnectionTimeout
     * 示例 URL: http://localhost:8080/record-error?type=validation&msg=InvalidEmail
     */
    @GetMapping("/record-error")
    public String recordError(
            @RequestParam(defaultValue = "unknown") String type,
            @RequestParam(defaultValue = "generic") String msg) {
        
        metricService.recordError(type, msg);
        
        return "Error count recorded successfully for type: " + type + ", message: " + msg;
    }
    
    /**
     * 访问此接口来触发一个成功的调用 (不计数)
     */
    @GetMapping("/success")
    public String success() {
        return "Success operation.";
    }
}
```

### 5\. 启动类：`src/main/java/com/example/demo/DemoApplication.java`

标准的 Spring Boot 启动类。

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

-----

### 🚀 运行与验证

1.  **运行项目**：在 IDE (如 IntelliJ IDEA) 中运行 `DemoApplication.java` 的 `main` 方法，或使用 Maven 命令 `mvn spring-boot:run`。
2.  **触发计数**：在浏览器或命令行中多次访问以下 URL，以模拟不同类型的错误：
    * `http://localhost:8080/record-error?type=database&msg=ConnectionTimeout`
    * `http://localhost:8080/record-error?type=validation&msg=InvalidEmail`
    * `http://localhost:8080/record-error?type=database&msg=ConnectionTimeout` (再访问一次)
3.  **验证指标暴露**：访问 Prometheus 端点，检查指标数据是否已生成：
    * `http://localhost:8080/actuator/prometheus`

您应该能在输出中找到类似以下格式的数据（注意名字变成了 `account_error_count_total`）：

```
# TYPE account_error_count_total counter
account_error_count_total{error_msg="ConnectionTimeout", tag_type="database"} 2.0
account_error_count_total{error_msg="InvalidEmail", tag_type="validation"} 1.0
```

现在，我们有了实际运行的数据。下一步就是 **配置 Grafana 连接到这个数据源并创建图表**


这是一个非常重要的步骤，我们将配置 Grafana 和 Prometheus，把你的 Spring Boot 应用真正连接到可视化界面。

要实现这个目标，我们必须引入中间层 **Prometheus**，它负责定时去你的 Spring Boot 应用（`http://localhost:8080/actuator/prometheus`）拉取（Scrape）数据，然后 Grafana 再从 Prometheus 获取这些时序数据。

### 架构概览

$$\text{Spring Boot (Metrics Source)} \xrightarrow{\text{Scrape at 8080/actuator/prometheus}} \text{Prometheus (Time-Series DB)} \xrightarrow{\text{Query}} \text{Grafana (Visualization)}$$

假设你已经安装了 Docker，以下是配置和查询的详细步骤。

-----

## 步骤一：配置 Prometheus 采集目标

我们需要告诉 Prometheus 服务器，你的 Spring Boot 应用在哪里。

### 1\. 创建 Prometheus 配置文件 (`prometheus.yml`)

在您的工作目录下创建一个名为 `prometheus.yml` 的文件，内容如下。这个配置将 Prometheus 服务器设置为每隔 5 秒去采集您的 Spring Boot 应用的指标。

```yaml
global:
  # 每隔 5 秒评估一次规则和采集数据
  scrape_interval: 5s 

scrape_configs:
  # Prometheus 自身的监控
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090'] # Prometheus 默认端口
      
  # 您的 Spring Boot 应用监控
  - job_name: 'spring-boot-app'
    # 假设您的应用运行在本地 8080 端口
    static_configs:
      - targets: ['host.docker.internal:8080'] 
    # 指标路径，Actuator 默认暴露在这个路径
    metrics_path: '/actuator/prometheus'
```

> **注意：** `host.docker.internal` 是 Docker 容器内部访问宿主机 `localhost` 的特殊地址。

## 步骤二：启动 Prometheus 和 Grafana

为了快速搭建环境，我们使用 Docker Compose 启动 Grafana 和 Prometheus。

### 2\. 创建 Docker Compose 文件 (`docker-compose.yml`)

```yaml
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      # 挂载我们上面创建的配置文件
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    networks:
      - monitoring-net

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    networks:
      - monitoring-net
    depends_on:
      - prometheus
      
networks:
  monitoring-net:
    driver: bridge
```

### 3\. 运行容器

确保你的 Spring Boot 应用正在运行（端口 `8080`），然后在 `prometheus.yml` 和 `docker-compose.yml` 所在的目录运行：

```bash
docker-compose up -d
```

现在，环境已准备就绪：

* **Spring Boot App:** `http://localhost:8080`
* **Prometheus UI:** `http://localhost:9090`
* **Grafana UI:** `http://localhost:3000` (默认登录：`admin`/`admin`)

## 步骤三：在 Grafana 中添加数据源

登录 Grafana (默认: `admin/admin`)，然后执行以下操作：

1.  在左侧导航栏，点击 **Settings** (齿轮图标) -\> **Data Sources**。
2.  点击 **Add data source**，选择 **Prometheus**。
3.  在 **HTTP** 部分，填写 Prometheus 服务器的 URL：
    * **URL:** `http://prometheus:9090` (因为 Grafana 和 Prometheus 在同一个 Docker 网络中，可以直接使用服务名)
4.  点击底部的 **Save & Test**。如果看到 "Data source is working" 的提示，说明连接成功。

## 步骤四：创建仪表盘和可视化

现在我们来创建图表，展示你的 `account.error.count` 指标。

1.  在左侧导航栏，点击 **Dashboards** (四个方块图标) -\> **New Dashboard** -\> **Add visualization**。

2.  选择你刚刚配置好的 **Prometheus** 数据源。

3.  在 **Query** (查询) 区域，输入你的 PromQL 查询语句：

    | 目标 | PromQL 查询语句 | 说明 |
        | :--- | :--- | :--- |
    | **总错误计数 (Raw Counter)** | `account_error_count_total` | 显示当前累积的总错误次数。|
    | **每分钟新增错误数 (Rate)** | `rate(account_error_count_total[1m])` | 这是更常用的查询方式。它计算你的计数器在过去 1 分钟内的平均增长率（即每秒新增的错误数）。|
    | **特定类型错误计数** | `sum by (tag_type) (account_error_count_total)` | 按错误类型 (`tag_type`) 分组求和，可以清楚看到哪种错误最多。|
    | **过滤特定错误** | `rate(account_error_count_total{tag_type="database"}[1m])` | 仅显示 `tag_type` 为 `database` 的错误增长率。|

4.  选择一个你喜欢的可视化类型，例如 **Graph** 或 **Stat**。

5.  在 Spring Boot 应用中，多访问几次你创建的 `/record-error` 接口来生成数据。

6.  在 Grafana 中，你就能看到数据图表开始实时绘制！

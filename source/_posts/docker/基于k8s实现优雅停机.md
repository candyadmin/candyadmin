---
title: 基于k8s实现优雅停机
date: 2025-06-06 12:27:16
categories: k8s
tag: chrome
---
在 Kubernetes 中，**默认情况下 pod 会在接收到 `SIGTERM` 信号（例如升级、删除等场景）后，立即从 Service 的负载列表中摘除（Endpoint 移除）**，然后再执行 `preStop` 钩子以及进入 `terminationGracePeriodSeconds` 等步骤。

你说的需求是：

> **在发版时，Pod 收到停止信号后，不要立即摘除它的 IP 和端口（即不立即从服务列表中剔除），而是等到你主动完成一些操作（如优雅停机通知）后，再摘除。**

这个需求在 Kubernetes 中可以通过以下几种方式实现：

---

## ✅ 实现目标：控制 Pod 在 `SIGTERM` 时 **延迟摘除 IP 和端口**

---

## 🧩 方法一：使用 `terminationGracePeriodSeconds` + 延迟摘除逻辑

Kubernetes 的**摘除 Endpoint 行为**是这样的：

* Pod 一旦进入 Terminating 状态，**会立即从 Service 的 endpoints 中移除**
* 并不是等 `preStop` 或 `terminationGracePeriodSeconds` 完成后才移除

因此想“暂时保留 IP 不被摘除”——**需要规避 Pod 直接进入 Terminating 状态**

### 🧠 核心方案：**控制流量的移除时机，而不是控制 Pod 的下线时机**

---

## ✅ 方法二：使用 \[**Readiness Probe**] 控制流量摘除时机（推荐）

### 步骤如下：

1. Pod 启动时设置 `readinessProbe`，只有探针通过时才参与服务
2. 停机前，在 `preStop` 阶段**手动设置为 NotReady**（流量会被摘除，但 Pod 仍在）

---

### ✅ 实现方式（推荐）

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### ⚙️ preStop 阶段“手动设置为不健康”

```yaml
lifecycle:
  preStop:
    exec:
      command:
        - /bin/sh
        - -c
        - |
          echo "Notifying service to stop sending traffic"
          curl -X POST http://localhost:8080/shutdown-hook
          sleep 30  # 等待流量转移 + 优雅停机
```

在 Spring Boot 中可以结合 actuator 管理健康状态：

```java
@RestController
public class ShutdownController {

    private final ApplicationAvailability availability;

    @Autowired
    public ShutdownController(ApplicationAvailability availability) {
        this.availability = availability;
    }

    @PostMapping("/shutdown-hook")
    public void shutdown() {
        // 手动设置为不可用（也可以切换 readiness probe 的状态）
        // 若使用 spring-boot-actuator 2.3+ 可切换 Readiness
        System.out.println("Application is preparing to shutdown...");
    }
}
```

---

## ❗ 为什么不能“阻止 IP 被摘除”

K8s 的 Endpoint Controller 一旦发现 Pod 是 Terminating 状态（即使你还在 `preStop` 和 `terminationGracePeriodSeconds` 里），就会从 Service 中移除 Endpoint（也就是从 DNS 或网关负载中摘除 IP）。

> 所以，要想“发版时不立即摘掉 IP”，你要避免一开始就让 Pod 进入 Terminating，而是**先标记为 NotReady**，等处理完毕再删除 Pod。

---

## ✅ 总结推荐配置

| 项目                               | 推荐配置                                   |
| -------------------------------- | -------------------------------------- |
| `readinessProbe`                 | 用于控制服务是否接受流量                           |
| `preStop` hook                   | 优雅通知微服务关闭，并 sleep 等待                   |
| `terminationGracePeriodSeconds`  | 设置为 ≥ `preStop` 脚本耗时，例如 60s            |
| `Spring Boot readiness endpoint` | 建议使用 actuator + readiness 实现接口健康状态动态切换 |

---

### 💡 示例完整配置片段

```yaml
spec:
  containers:
    - name: trade
      image: trade:latest
      lifecycle:
        preStop:
          exec:
            command:
              - /bin/sh
              - -c
              - |
                curl -X POST http://localhost:8080/shutdown-hook
                sleep 30
      readinessProbe:
        httpGet:
          path: /actuator/health/readiness
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
  terminationGracePeriodSeconds: 60
```

---

如你想结合 Spring Boot actuator 做动态健康状态切换（比如调用 `/actuator/health/readiness` 返回 `DOWN` 状态）。


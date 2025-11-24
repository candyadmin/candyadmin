---
title: linux下修复网卡出入口策略
date: 2025-10-17 17:51:59
tags:
---
2️⃣ 查看路由表
    ip route show
```shell
default via 172.18.8.1 dev enp4s0 proto dhcp src 172.18.8.14 
default via 172.18.4.1 dev wlx90de8039f84e proto dhcp src 172.18.5.84 metric 600 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
172.18.0.0/16 dev br-1d60fb2053b6 proto kernel scope link src 172.18.0.1 linkdown 
172.18.4.0/22 dev wlx90de8039f84e proto kernel scope link src 172.18.5.84 metric 600 
172.18.4.53 dev enp4s0 scope link 
172.18.8.0/24 dev enp4s0 proto kernel scope link src 172.18.8.14 
172.19.0.0/16 dev br-2ef4d7ca1b69 proto kernel scope link src 172.19.0.1 
172.20.0.0/16 dev br-5f32771043be proto kernel scope link src 172.20.0.1 linkdown 
172.21.0.0/16 dev br-ad6eae2e33ba proto kernel scope link src 172.21.0.1 linkdown 
172.22.0.0/16 dev br-27852fab7433 proto kernel scope link src 172.22.0.1 linkdown 
172.23.0.0/16 dev br-d47499b873f8 proto kernel scope link src 172.23.0.1 linkdown 
172.24.0.0/16 dev br-26459e3bb6fd proto kernel scope link src 172.24.0.1 linkdown 
172.25.0.0/16 dev br-b134fb0df423 proto kernel scope link src 172.25.0.1 linkdown 
172.26.0.0/16 dev br-f90da39cf0eb proto kernel scope link src 172.26.0.1 
172.27.0.0/16 dev br-1688b9c0bb44 proto kernel scope link src 172.27.0.1 
172.28.0.0/16 dev br-1e43fea341a0 proto kernel scope link src 172.28.0.1 linkdown 
192.168.1.0/24 dev virbr1 proto kernel scope link src 192.168.1.1 linkdown 
```

我们可以看出你的系统现在有两个「默认路由（default route）」，并且它们走的是不同的网卡：

---

## 🧭 当前路由情况分析

| 项                     | 内容                                                      |
| --------------------- | ------------------------------------------------------- |
| 有线网卡（enp4s0）          | IP: **172.18.8.14**，网关：**172.18.8.1**                   |
| 无线网卡（wlx90de8039f84e） | IP: **172.18.5.84**，网关：**172.18.4.1**                   |
| 默认路由1                 | `default via 172.18.8.1 dev enp4s0`                     |
| 默认路由2                 | `default via 172.18.4.1 dev wlx90de8039f84e metric 600` |

---

## 🧠 问题根源

系统现在有两条默认出口：

* **优先级较高**：`enp4s0`（因为没有 metric，默认 metric=0）
* **优先级较低**：`wlx90de8039f84e`（metric 600）

这意味着：

* 请求从两个网卡都能进来；
* 但所有响应包（默认出口）会优先走 **enp4s0**；
* 所以如果入口是无线（wlx90de8039f84e），包出去却从有线（enp4s0）走，就会出现“入口出口不一致”的情况（典型问题）。

---

## 🔧 修复方案一：让出口和入口一致（策略路由）

### 适用情况

如果你希望：

> 哪个网卡收到请求，就从哪个网卡回去。

### 具体操作

1️⃣ 创建两个策略路由表

```bash
echo "100 enp4s0" >> /etc/iproute2/rt_tables
echo "200 wlx90de8039f84e" >> /etc/iproute2/rt_tables
```

2️⃣ 添加规则与路由

```bash
# 有线网卡 enp4s0
ip rule add from 172.18.8.14/32 table enp4s0
ip route add default via 172.18.8.1 dev enp4s0 table enp4s0

# 无线网卡 wlx90de8039f84e
ip rule add from 172.18.5.84/32 table wlx90de8039f84e
ip route add default via 172.18.4.1 dev wlx90de8039f84e table wlx90de8039f84e
```

3️⃣ 验证规则

```bash
ip rule show
ip route show table enp4s0
ip route show table wlx90de8039f84e
```

4️⃣ 测试是否从相同网卡返回：

```bash
ping -I 172.18.8.14 8.8.8.8   # 走有线
ping -I 172.18.5.84 8.8.8.8   # 走无线
```

✅ 这样就能保证：

* 从哪个网卡进来的连接，返回包也从那个网卡出去；
* 不再出现“入口出口不一致”的情况。

---

## 🔧 修复方案二：强制统一出口网卡（简单粗暴）

如果你只希望所有流量都走某个网卡（比如有线），可直接修改默认路由：

```bash
ip route del default
ip route add default via 172.18.8.1 dev enp4s0
```

可选：设置 `metric`，避免被 DHCP 自动添加的路由覆盖：

```bash
nmcli connection modify enp4s0 ipv4.route-metric 100
nmcli connection modify wlx90de8039f84e ipv4.route-metric 600
```

这样系统会始终优先用有线网卡出网。

---

## 💾 永久生效方法

不同系统略有差异：

* **Ubuntu/Debian**：在 `/etc/network/interfaces` 或 `/etc/netplan/*.yaml` 里添加 `post-up` 命令；
* **RHEL/CentOS**：放到 `/etc/sysconfig/network-scripts/route-enp4s0`；
* 或写入 `/etc/rc.local` 并 `chmod +x`。

---

## 📊 总结建议

| 目标        | 推荐方案                          |
| --------- | ----------------------------- |
| 自动匹配入口出口  | 策略路由（方案一 ✅）                   |
| 全部流量走一个网卡 | 修改默认路由（方案二）                   |
| 临时测试      | `ip rule` / `ip route` 手动执行即可 |

---

是否方便告诉我：
👉 你的目标是「**所有流量都走一个出口**」，还是「**哪个网卡进就从哪个网卡出**」？
我可以帮你写出**最简洁、可直接复制的最终命令**。

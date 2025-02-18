---
title: linux下通过端口找到进程
date: 2024-08-09 14:48:30
categories: linux
tag: 查找进程
---
在Linux系统中，可以使用以下几种方法通过端口号查找占用该端口的进程：

### 1. 使用 `lsof` 命令
`lsof` 是一个用于列出打开的文件的命令，它也可以用于查找正在使用某个端口的进程。

```bash
lsof -i :<端口号>
```

例如，查找占用80端口的进程：
```bash
lsof -i :80
```

### 2. 使用 `netstat` 命令
`netstat` 是一个网络状态监控工具，可以显示与网络相关的内容，包括哪些进程在监听哪些端口。

```bash
netstat -tuln | grep :<端口号>
```

例如，查找占用80端口的进程：
```bash
netstat -tuln | grep :80
```

结合 `netstat` 和 `grep` 命令，还可以找到对应的PID：
```bash
netstat -tulnp | grep :<端口号>
```

### 3. 使用 `ss` 命令
`ss` 是 `netstat` 的替代品，用于显示socket统计信息，可以更快速地显示网络连接。

```bash
ss -tuln | grep :<端口号>
```

例如，查找占用80端口的进程：
```bash
ss -tuln | grep :80
```

### 4. 使用 `fuser` 命令
`fuser` 是一个显示哪个进程使用哪些文件或socket的命令。可以直接显示占用指定端口的进程PID。

```bash
fuser <端口号>/tcp
```

例如，查找占用80端口的进程：
```bash
fuser 80/tcp
```

### 5. 查看详细的进程信息
一旦获得了PID，可以使用 `ps` 命令来查看进程的详细信息。

```bash
ps -p <PID> -o pid,cmd
```

例如：
```bash
ps -p 1234 -o pid,cmd
```

这将显示PID为1234的进程的命令行。

通过这些方法，您可以很容易地查找到占用特定端口的进程并获取相应的PID和其他信息。
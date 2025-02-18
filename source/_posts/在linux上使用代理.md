---
title: 在linux上使用代理
date: 2024-07-26 09:56:32
categories: clash
tag: 翻墙
---
要在 Linux 命令行下使用 Clash Verge，需要按照以下步骤进行：

### 1. 安装 Clash Verge

首先，确保你的系统已经安装了 Clash Core。Clash Verge 是一个图形界面管理工具，而 Clash 本身是需要运行在后台的代理工具。

#### 下载 Clash

[mihomo-linux-amd64-v1.18.6.deb](在linux上使用代理/mihomo-linux-amd64-v1.18.6.deb)
```bash
dpkg -i mihomo-linux-amd64-v1.18.6.deb     
chmod 777 mihomo-linux-amd64-v1.18.6.deb 
sudo mv mihomo-linux-amd64-v1.18.6 clash
```

### 2. 配置 Clash
在使用 Clash Verge 之前，需要先配置 Clash 的 `config.yaml` 文件。

```bash
拷贝windows上的配置：D:\soft\clash-verge-1.6.2\.config\io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml
wget -O /root/.config/mihomo/config.yaml 你配置好的clash-verge.yaml
```
### 3. 启动 Clash
在命令行中启动 Clash：
```bash
nohup ./clash &
```
Clash Verge 启动后，将打开一个本地的图形界面，你可以通过浏览器访问该界面，通常是 `http://localhost:7899` 或者其他配置文件中指定的端口。



1. **创建代理配置文件**：在Kali系统的Docker配置目录下创建一个名为`/etc/systemd/system/docker.service.d/http-proxy.conf`的文件。如果该目录不存在，请先创建它。

```sh
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf
```

2. **配置代理**：在`http-proxy.conf`文件中添加以下内容，将`http://127.0.0.1:7899`替换为你的代理服务器地址和端口。

```ini
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7899"
Environment="HTTPS_PROXY=http://127.0.0.1:7899"
Environment="NO_PROXY=localhost,127.0.0.1"
```

3. **重新加载守护进程并重启Docker服务**：

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

4. **验证代理配置**：你可以通过以下命令来验证Docker是否通过代理服务器进行连接：

```sh
docker info
```

在输出信息中找到`HTTP Proxy`和`HTTPS Proxy`条目，检查它们是否显示了你配置的代理服务器地址。

如果你需要为Docker客户端配置代理（如`docker build`命令），可以在你的shell配置文件（如`~/.bashrc`或`~/.zshrc`）中添加以下行：

```sh
export HTTP_PROXY="http://your-proxy.example.com:8080"
export HTTPS_PROXY="http://your-proxy.example.com:8080"
export NO_PROXY="localhost,127.0.0.1"
```

保存文件并加载新的配置：

```sh
source ~/.bashrc  # 或者 source ~/.zshrc
```

这样，Docker客户端在运行时也会通过代理服务器进行连接。
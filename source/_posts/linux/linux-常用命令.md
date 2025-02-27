---
title: linux-常用命令
date: 2025-02-20 17:47:37
categories: linux
tag: 常用命令
---

linux自启动：

```bash
cd /etc/systemd/system/
vi openvpn-client.service
```

```bash
[Unit]
Description=OpenVPN client service
After=network.target
 
[Service]
Type=simple
ExecStart= /usr/sbin/openvpn --config /opt/openvpn/huawei-5600g.ovpn
Restart=always
 
[Install]
WantedBy=multi-user.target
```
```bash
systemctl daemon-reload
systemctl enable openvpn-client.service
```
添加jdk环境变量：
```bash
　编辑/etc/profile文件 vim /etc/profile    <<---- 通过这种方式，在关闭xshell后，添加的环境变量不生效

　　文件末尾添加：export PATH="/usr/local/nginx/sbin/:$PATH"

　　source  /etc/profile
```
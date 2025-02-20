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

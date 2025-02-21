---
title: openvpn-部署脚本
date: 2024-08-07 16:02:35
categories: linux
tag: openvpn
---
1.执行安装脚本：
[openvpn.sh](openvpn-部署脚本/openvpn.sh)

2.将覆盖的dns等配置注释掉
``` server
local 192.168.2.124
port 5001
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
auth SHA256
tls-crypt tc.key
topology subnet
server 10.8.0.0 255.255.255.0
#push "block-ipv6"
#push "ifconfig-ipv6 fddd:1194:1194:1194::2/64 fddd:1194:1194:1194::1"
#push "redirect-gateway def1 ipv6 bypass-dhcp"
ifconfig-pool-persist ipp.txt
#push "dhcp-option DNS 100.125.1.250"
#push "block-outside-dns"
keepalive 10 120
cipher AES-128-GCM
user nobody
group nogroup
persist-key
```
---
layout: post
title: "wireguard"
subtitle: "postgresql 学习"
date: 2021-09-17 09:50:00
author: "wangdiqi"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - vpn
---

> "Let's go"

# 常用命令

## 服务端

参考官方教程安装wireguard：https://www.wireguard.com/install/

~~~
1. 生成private key
wg genkey > private-key

2. 查看public key
wg pubkey < ./private-key

3. 创建并编辑wg.ini
[Interface]
ListenPort = 13004
PrivateKey = xxxxxx

4. 创建interface
sudo ip link add dev wg0 type wireguard

5. 给interface绑定地址
sudo ip addr add 10.0.5.1/24 dev wg0

6. 启用interface
sudo ip link set wg0 up

7. 更新wg配置
sudo wg setconf wg0 ./wg.ini

8. 启用ip forward
echo 1 > /proc/sys/net/ipv4/ip_forward

9. nat
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

10. 网卡转发规则
sudo iptables -A FORWARD -i wg0 -j ACCEPT

11. 根据客户端信息，修改wg.ini,新增peer信息后重新载入配置
[Peer]
PublicKey = xxxxxxxxxxxxx
AllowedIPs = 10.0.5.2/32
~~~

## 客户端

参考官方教程安装wireguard：https://www.wireguard.com/install/

[Interface]
PrivateKey = xxxxxxxx
Address = 10.0.5.2/24
DNS = 1.1.1.1, 1.0.0.1

[Peer]
PublicKey = xxxxxxxx
AllowedIPs = 0.0.0.0/0
Endpoint = 服务端监听的端口和地址

---
title: 树莓派热点
date: 2021-03-11
updated: 2021-10-29
toc: true
mathjax: false
categories:
   - [树莓派]
tags:
   - hostapd
   - dnsmasq
   - Access Point
---

> 简单介绍如何在树莓派开启热点  
> *参考自[Setting up a routed wireless access point](https://www.raspberrypi.org/documentation/computers/configuration.html#setting-up-a-routed-wireless-access-point)*

需要的软件列表：

- `hostapd`：用于开启热点
- `dnsmasq`：用于开启DHCP服务与DNS服务
- `iptables`：设置数据包转发规则
- `netfilter-persistent`：将 `iptables` 设置的规则持久化
- `iptables-persistent`：`netfilter-persistent` 插件

### 安装软件

运行命令

```shell
# apt-get install hostapd dnsmasq iptables netfilter-persistent iptables-persistent
```

等待安装完成

### 配置hostapd

编辑配置文件 `/etc/hostapd/hostapd.conf`，添加配置

```conf
# 使用的网卡
interface=wlan0
# WiFi 名字
ssid=YOUR_WIFI_NAME
# 驱动
driver=nl80211
# WiFi 频率模式
hw_mode=g
# 信道
channel=7
# 是否 mac 过滤
macaddr_acl=0
# 认证算法
auth_algs=1
# 是否广播 SSID
ignore_broadcast_ssid=0
# 加密相关
wpa=2
wpa_passphrase=YOUR_WIFI_PASSWORD
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

**警告**:
channel 不可随意选择，不同的国家与地区允许使用的信道不同  

注：上面的配置启用的为 2.4GHz 的WiFi
如需使用 5GHz 的 WiFi，修改以下配置：

```conf
#hw_mode=g
hw_mode=a
#channel=7
# 在内核启 CONFIG_ACS 编译选项后
# 该选项设置为 0 时会自动选择信道
# 树莓派似乎不支持......
channel=149
```

### 配置dhcpcd

编辑配置文件 `/etc/dhcpcd.conf`，添加配置

```conf
# 网卡
interface=wlan0
# 网卡地址
static ip_address=192.168.10.1/24
# 不要执行这个脚本 
nohook wpa_supplicant
```

### 配置dnsmasq

编辑配置文件 `/etc/dnsmasp.conf`

```conf
# 为这个网卡开启服务
interface=wlan0
# 不要在这个网卡开启服务
no-dhcp-interface=eth0
# 本机域
domain=wlan
# IP 地址分配范围
dhcp-range=192.168.10.1,192.168.10.20,255.255.255.0,24h
```

### 设置数据包转发

1. 允许包转发  
   编辑配置文件 `/etc/sysctl.conf`  
   将 `net.ipv4.ip_forward=1` 取消注释

2. 设置转发规则
   运行命令

   ```shell
   # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
   ```

3. 保存规则
   运行命令

   ```shell
   # netfilter-persistent save
   ```

### 设置systemd启动服务

启用 `hostapd.service`，`dnsmasq.service`，`netfilter-persistent.service`  
禁用 `wpa_supplicant.service`

```shell
# systemctl unmask hostapd.service
# systemctl enable hostapd.service dnsmasq.service netfilter-persistent.service
# systemctl disable wpa_supplicant
```

### 重启

设置完成后重启

```shell
# reboot
```

不出意外，就能在 WiFi 列表里找到树莓派开启的 WiFi 了

## 参考链接

[hostapd.conf 设置说明](https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf)  
[维基百科：WLAN channel](https://en.wikipedia.org/wiki/List_of_WLAN_channels)

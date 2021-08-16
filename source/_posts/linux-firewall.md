---
title: Linux 防火墙
toc: true
mathjax: false
categories:
  - - 笔记
  - - Linux
tags:
  - Firewall
date: 2021-02-18 23:04:07
---

> 本文仅做简单的介绍，不涉及防火墙的实际应用及操作

## Netfilter Porject

Netfilter 项目是一个开源项目，目的是为 Linux 内核提供进行包过滤的功能

> 官网[Netfilter Project](https://www.netfilter.org)介绍原文:
> "*The netfilter project is a community-driven collaborative FOSS project that 
> provides packet filtering software for the Linux 2.4.x and later kernel series.
> The netfilter project is commonly associated with iptables.*"

## 四表五链概念

> 参考 `man iptables`，以及网络资料

存在 `表(tables)`、`链(chain)` 和 `规则(rules)` 三个层面
- `表(tables)` 指的是不同类型的数据包处理流程，`表` 中可以存在多个 `链`
- `链(chain)` 中包含若干 `规则`
- `规则(rules)` 规定数据包的处理方式
系统接收到数据包时将根据 `链` 中的 `规则` 将包通过 `链`

### **四表**
四表用于管理链路规则
| 表名     | 作用                                       |
|:---------|:------------------------------------------|
| Filter表 | 过滤数据包                                  |
| NAT表    | 用于网络地址转换                             |
| Mangle表 | 修改数据包服务类型、TTL、并且可以配置路由实现QOS |
| Raw表    | 决定数据包是否被状态跟踪机制处理               |

### **五链**
五条链用于存放链路规则
| 链名           | 作用对象           |
|:---------------|:-----------------|
| INPUT链        | 进来的数据包        |
| OUTPUT链       | 发出的数据包        |
| FORWARD链      | 转发的数据包        |
| OREROUTING链   | 作路由选择前的数据包 |
| POSTROUNTING链 | 作路由选择后的数据包 |

### **四表五链之间的联系**
四表里存放的是功能一致的规则，即存放着链
表中存放的规则如下
| Filter表 |     NAT表    |   Mangle表   |     Raw表    |
|:---------|:-------------|:-------------|:-------------|
| INPUT链  | OUTPUT链     | INPUT链      | OUTPUT链     |
| OUTPUT链 | PREROUTING链 | OUTPUT链     | PREROUTING链 |
| FORWARD链| POSTROUTING链| FORWARD链    |              |
|          |              | PREROUTING链 |              |
|          |              | POSTROUTING链|              |

## 管理 firewall

下面列出的是运用最广泛的防火墙管理软件，以及他们之间的联系
- `netfilter` 是一个 Linux 内核的软件框架，用于管理网络数据包[^netfilter]，运行在内核空间
- `iptables` 是运行在用户空间的软件，通过控制 netfilter 模块来管理网络数据包的处理和转发[^iptables]
- `nftables` 是 iptables 的后继者[^nftables]
- `uncomplicated ftrewall(UFW)` 的使用更加简单方便，依赖于 iptables 来管理网络包[^UFW]
- `firewalld` 作为 nftables 的前端，并通过其来管理网络包，也可以一使用 iptables[^firewalld]

[^netfilter]: 参考: [维基百科 Netfilter](https://zh.wikipedia.org/wiki/Netfilter)
[^iptables]: 参考: [维基百科 Iptables](https://zh.wikipedia.org/wiki/Iptables)
[^nftables]: 在 Netfilter 项目官网主页指出: *nftables is the successor of iptables*
[^UFW]: 参考: [维基百科 Uncomplicated Firewall](https://en.wikipedia.org/wiki/Uncomplicated_Firewalld)
[^firewalld]: 参考: [维基百科 Firewalld](https://en.wikipedia.org/wiki/Firewalld)


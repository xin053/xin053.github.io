---
title: Docker学习笔记(二)
date: 2016-09-20 16:04:51
categories: 
- Docker
tags:
- Docker
---

## 解决问题
日常升级完VirtualBox后，发现打开docker虚拟机出差，报错：

```
Failed to open/create the internal network 'HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter'
```

解决方法：

- 打开网络和共享中心
- 更多适配器设置
- 选择 对应的网络适配器 adapter （如： VirtualBox Host-Only Ethernet Adapter）
- 右键“属性”
- 然后勾选 VirtualBox NDIS6 Bridged Networking Driver 选项，确定

<!-- more -->



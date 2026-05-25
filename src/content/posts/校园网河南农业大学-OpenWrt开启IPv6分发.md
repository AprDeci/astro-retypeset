---
title: "校园网(河南农业大学) Openwrt 开启IPV6分发"
published: 2024-02-28
tags:
  - openwrt
  - nas
  - campus-network
draft: false
toc: true
lang: zh
---
校园网(河南农业大学) Openwrt 开启IPV6分发

因为在家里放着NAS,不论是DDNS还是虚拟组网,有IPV6对访问的速度和稳定性都有极高的提升,所以尝试寝室网分发IPV6.一般大学都有IPV6

[河南省教育系统IPv6发展监测平台 (ha.edu.cn)](https://ipv6.ha.edu.cn/monitor.html)

不废话直接看图设置![image-20240228115314718](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240228115314718.png)

![image-20240228115335300](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240228115335300.png)

![image-20240228115347523](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240228115347523.png)

![image-20240228115403003](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240228115403003.png)

Openwrt对应禁止解析IPV6

![image-20240228115441774](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240228115441774.png)

一般设置到这里就会分发了
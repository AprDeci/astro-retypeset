---
title: "tailscale两台openwrt双lan网对网site-to-site互通"
published: 2024-09-02
tags:
  - openwrt
  - tailscale
  - github
draft: false
pin: 1
toc: true
lang: zh
---
# tailscale两台openwrt(双lan)网对网(site to site)互通

**环境:家中路由器 lan:192.168.8.0/24,ip:192.168.8.1**

**寝室路由器 lan:192.168.17.0/24,ip:192.168.17.1前置准备**

site to site可以使两个局域网的设备无需安装tailscale即可通过lan地址访问其他局域网的设备

## 前置准备

一些路由器系统由于nft软件包安装的不完整,导致tailscale防火墙规则不完整(点名istoreos),需要去软件包中搜索**"nft"**,将你能安装的都安装了(其实我也不知道具体要安装哪几个,希望大佬指教)

## 设置子网路由(两台路由器均执行)

先讲思路:为每一个openwrt设备开启子网路由->为openwrt设备设置静态路由

对!思路就这么简单.

安装软件:[CH3NGYZ/tailscale-openwrt: tailscale on openwrt 一键部署脚本 (github.com)](https://github.com/CH3NGYZ/tailscale-openwrt)

安装完成后输入tailscale up(部署脚本应该会提示让你登录,登陆完无需此步.)

查看tailscale控制台是否接入你的路由器设备.

路由器ssh输入(下方的192.168.x.0/24),修改为你当前路由的lan区域地址,譬如我的家中路由器是192.168.8.0/24,宿舍路由器是192.168.17.0/24

~~~shell
tailscale up --accept-routes --advertise-routes=192.168.8.0/24 --accept-dns=false
~~~

输入完后看控制台的设备下面有subnet!字符,点击右边...![image-20240902095027386](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240902095027386.png)为你的lan网区域打上勾.

### 接口配置

在openwrt 网络-接口 点击添加新接口,名称设置为tailscale0,协议静态地址,设备选择tailscale0,点击确定,然后编辑相关信息,IPV4地址填写控制台中tailscale分配的地址(一般是100.xx.xx.xx),子网掩码填写255.0.0.0.

防火墙设置为lan区域(自建一个tailscale区域也可以,效果都是一样的最终)

![image-20240902095448045](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240902095448045.png)

其他无需填写直接保存![image-20240902095407416](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240902095407416.png)

### 防火墙设置

如果你为tailscale单独设置了防火墙区域,前去区域设置为lan添加允许转发的目标区域添加tailscale,允许来自tailscale区域转发也添加上

(图片仅演示,我设置为lan区域)

![image-20240902095609413](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240902095609413.png)

入栈出栈转发均设置为接受.

## 添加静态路由

当你的两台路由器均设置了子网路由后(子网路由IP地址不能一致且和你的现实IP地址保持一致)

(两台路由器)在openwrt 网络->路由->静态ipv4路由中添加如下

**网关是当前路由器设备地址,目标是对方局域网地址(下图为宿舍路由器添加路由)**

![image-20240902095834924](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240902095834924.png)

保存后就可以互通了.

tailscale还让添加一个tailscale地址的静态路由,我感觉没啥影响对我的应用场景,如果你需要添加,则按如下添加(两个路由器填写一致)

![image-20240902100234979](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240902100234979.png)

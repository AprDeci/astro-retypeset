---
title: "openwrt Zerotier设置"
published: 2023-07-13
tags:
  - openwrt
  - zerotier
draft: false
toc: true
lang: zh
---
# openwrt Zerotier设置

今天早上起床发现系统盘突然爆满.让我一脸懵逼.于是干脆重新刷机换了iStoreOS,自己去下载需要的软件,上一个用的系统(不说是哪个大佬整合的了)太乱了,很多软件兼容都没搞好.

## 安装插件包

这个应该很多系统自带Zerotier了,如果没有可以用下面的命令来安装

~~~bash
opkg update
opkg install zerotier
~~~

~~~bash
https://op.supes.top/packages/
~~~

~~~bash
opkg install luci-app-zerotier_*.ipk
~~~

输入完上面的命令后,应该就可以在Vpn子菜单看到Zerotier了

## 申请地址

打开

[Zerotier]: www.zerotier.com	"Zerotier官网"

注册个账号进入控制台(右上角Networks),点击Create A Network,你就会发现下面多了一个网络组,就好了.

## 网络组设置

Access Control 选择private

在IPv4 Auto-Assign中选择一个前24位都确定的地址,譬如![image-20230626170344530](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626170344530.png)

其他也没什么设置得了.

## Openwrt设置

在Zerotier的设置界面,填入网络ID,全部打钩

![image-20230626170508135](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626170508135.png)

***还需要在后台网络中设置防火墙***

首先在网络-接口中添加新的接口,协议是DHCP客户端,设备是Zxxxxxxx(可能是Z开头吧)一串字母的,就是Zerotier的网络接口

![image-20230626170723914](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626170723914.png)

添加后设置防火墙(有的添加后可以直接设置,有的得去接口页面单独编辑防火墙,道理都一样)

因为我是河南农业大学的校园网,选wan(是lan是wan不确定的自己试试就好了)

![image-20230626170908242](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626170908242.png)

接着在网络-防火墙中添加区域设置

![image-20230626171119444](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626171119444.png)

然后返回Zerotier的控制台,会看到用户多了软路由的信息,前面的对号点上,就可以分配IP了,然后在上面***Managed Routes***中添加链路,

![image-20230626171409268](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626171409268.png)

然后设置就结束了

## 结尾

你还可以去ss页面输入 看看是否成功分配

~~~bash
zerotier-cli listnetworks
~~~



![image-20230626171459847](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230626171459847.png)

至于自建moon和planet服务器,我自己试了试速度还不如人家自带的服务器呢~
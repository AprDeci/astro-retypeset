---
title: "nastools自动影视库"
published: 2023-07-13
tags:
  - nas
draft: false
toc: true
lang: zh
---
# nastools自动影视库

买来软路由第一时间就是用docker搭建了emby,并且买了一个硬盘盒来做媒体盘,之前一直是手动下载影片,昨天换了系统后想要尝试一下nastools,有docker几乎所有软件都可以安装,经过调试后,效果十分不错.
![image-20230627160817210](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230627160817210.png)



## 配置的坑

### 代理服务器

其实起初不知道nastools3.+版本需要验证用户解锁功能,安装好后,看了看验证用户的站点,正好有我在用的PT站..所以就直接通过验证了.

在设置里添加了TMDB的api,一开始设置了**代理服务器**,结果tmdb怎么也ping不通,信息加载不出来,苦恼了我一个晚上,在插件列表里添加hosts也不管用..

直到第二天,我把代理地址删了,才发现openclash是自动代理着本地的地址的,所以不需要额外配置....

### Emby链接

api好像名称必须是nastool 否则api会不识别,自己操作下吧

### 其他

目录同步要添加你的qb/transmission的下载路径,过滤规则选择日常观影设为默认即可

## tg推送设置

其实最眼馋的就是在微信/tg上远程下载电影的功能,一句消息就能自动下载电影并且配置完成,不觉得很酷吗.但是企业微信需要的条件太多,于是我选择了创建tg机器人.

方法十分简单,首先在

[Botfather]: https://t.me/BotFather

中创建一个机器人,跟随指引即可,然后获得api,在nastools-消息通知添加就好了,id什么的,看提示就懂啦.

于是我们实现了!tg机器人交互下载影片

![image-20230627161719830](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230627161719830.png)

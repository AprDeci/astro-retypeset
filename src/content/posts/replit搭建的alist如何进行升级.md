---
title: "replit搭建的alist如何进行升级"
published: 2023-01-06
tags:
  - github
draft: false
toc: true
lang: zh
---
# replit搭建的alist如何进行升级

alist可以用replit白嫖应该很多人都知道了,而且点进来的估计99%都是用replit搭建的,不得不说,体验相当好,可以一键搭建,不用什么复杂的方法.但是alist升级又成了一个问题.

**这个教程针对**

**[alist-org/alist-replit: alist on replit (github.com)](https://github.com/alist-org/alist-replit)**

**这个大佬发布的一键搭建.**

## 教程开始

其实升级十分简单,当然我也是自己摸索的,有什么更好的方法还请大佬留言告知.

replit启动就是根据文件里的main.sh文件的命令来进行的,

作者写的是

~~~bash
if[alist存在]

下载最新alist文件解压安装

fl

运行
~~~

(我不会linux命令,只是根据网上搜的理解的,有错误请指出)

所以我们搭建过一次后重启并不会再次下载,而是直接运行.

所以很简单,我们只需要把 

~~~bash
if[]

lf
~~~



删除,重启即可

经过本人测试数据什么的不会改变,只是单纯的覆盖了.![alist升级](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/alist升级.jpg)



**其实我觉得开机一次后一般也不会去关机或者重启什么操作,所以if判断完全可以删去,有更新的话重启更新一次即可,至少我是平时不会去关机重启什么的.**

修改后的如下:

~~~bash
# rm -rf alist* data/ #Uncomment this line to update
  curl -L https://github.com/alist-org/alist/releases/latest/download/alist-linux-musl-amd64.tar.gz -o alist.tar.gz
  tar -zxvf alist.tar.gz
  rm -f alist.tar.gz

./alist server --no-prefix

~~~




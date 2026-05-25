---
title: "Umami2.0VercelSupspace从1.4升级到2.2umami升级教程"
published: 2023-07-13
tags:
  - umami
draft: false
toc: true
lang: zh
---
# Umami2.0:Vercel+Supspace 从1.4升级到2.2(umami升级教程)

[toc]

## 前言

Umami最近大版本更新了,但是我太懒了,一直没升级,也一直在想要不要直接迁移到服务器上,省的Vercel+supsapce一直不方便.**而且umami没法统计到大陆用户的访问,是因为umami.js被墙了,大陆没办法访问到.**趁今天有时间,所幸看一看白嫖方案升级是否容易,顺便解决一下大陆访问问题.

想要搭建的可以看我之前写的[Umami1的白嫖教程](https://aprdec.top/index.php/archives/118)!

本文将解决:

1. Vercel Umami大版本升级
2. Umami js大陆访问(套CDN)

当然还会附带我在部署过程中碰到的一个BUG的解决方法.

## Umami大版本升级

(只针对Vercel等类似平台的升级,自己服务器的升级可以查看[Umami官方文档](https://umami.is),十分简单)

**不得不夸赞作者,升级真的是相当简单!**

**前提:电脑有Node.js,yarn环境**,这个是必须的,我尝试用Github的codespace代替,但是会出现爆栈问题.

Umami2的数据库结构发生了变化,由于在Vercel之类的托管平台没有权限,作者贴心的给出了数据库升级方案

~~~git
git clone https://github.com/umami-software/migrate-v1-v2.git 
cd migrate-v1-v2 
~~~

克隆了migratev1-v2的文件后,进入文件夹,使用命令行执行

~~~
yarn install
yarn build
~~~

然后在文件夹内创建**.env**文件,编辑内容如下:

**DATABASE_URL={connection url} ({}不需要)**

然后继续执行

~~~
yarn start
~~~

等待执行完毕,查看数据库可以发现数据库表发生变化<img src="https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/umami 迁移.jpg" alt="umami 迁移" style="zoom: 50%;" />

然后fork Umami2.2仓库,在Vercel里重新部署或搭建即可,搭建的过程和以前一样需要HASH_SALT以及DATABASE_URL的环境值.

**PS:2.0后Umami的静态文件从umami.js改成了script.js,需要重新修改**

## prepared statement 's0' already exits解决方法

这个在[umami的issue](https://github.com/prisma/prisma/issues/11643)里找到了解决方法,是我在部署Vercel时候碰到的bug![umami 出现问题](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/umami 出现问题.jpg)

解决方法就是在DATABASE_URL后面加?pgbouncer=true

例如:postgres://postgres:[YOUR-PASSWORD]@[host].supabase.co:6543/postgres?pgbouncer=true

## Umami js大陆访问(套CDN)

看的一个[博主的教程](https://roy.wang/umami-js-quicken/)

简单而言就是把umami给你的链接是一个js文件,下载下来,放到你的CDN中或者大陆能访问的服务器里,访问的时候访问你自己存储的文件就好了.

还得改一下跟踪代码,原来是这样的.

~~~javascript
<script async src="https://ver.aprdec.top/script.js"
data-website-id="25248b78-56df-4127-a7ca-093c3859fdf2">
    </script>
~~~



需要修改成

~~~javascript
<script async="" src="https://qiniu.aprdec.top/script.js" 
data-website-id="25248b78-56df-4127-a7ca-093c3859fdf2" 
data-host-url="https://ver.aprdec.top">
    </script>
~~~

如你所见,多了data-host-url的属性,该属性填写你的Umami的域名(不是CDN的域名),然后就可以出现五星红旗拉!

![umami2有了中国](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/umami2有了中国.jpg)
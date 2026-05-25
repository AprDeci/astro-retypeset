---
title: "搭建自己的相片备份服务器(immich)"
published: 2023-07-13
tags:
  - github
  - immich
draft: false
toc: true
lang: zh
---
# 搭建自己的相片备份服务器(immich)

[toc]

## 前言

今天在Github上看见一个好项目,是一个服务器相册备份的解决方案,看起来挺有意思操作起来也不难,并且配套了手机端和网页端,支持自动备份,我个人是比较看重相片备份的,毕竟有很多美好的回忆偶尔也可以看看,在我的夸克和阿里云盘都一直同步着我的相册,至今都有1000多张了.

看了看作者写的文档,服务器要求还是挺高的,**至少也要2h4g的配置**,我刚好有一台2h4g的美国服务器,数据盘有120G空着没用,正好试一试备份相册.

下面我来写搭建过程,写之前,我先把用到现在体验的优缺点说下.

优点

1. 搭建简单,配置简单
2. 能自动备份

缺点:

1. 服务器性能要求高
2. APP不能单独上传相片
3. APP优化不太好(可能跟我的网络环境有关)

## 搭建过程

作者写的文档:[IMMICH-Doc](https://immich-7e64pbtx0-immich.vercel.app/docs/install/docker-compose)

我是用的docker方式安装.

### 创建文件夹,下载文件

先创建文件夹,然后下载docker配置文件

~~~linux
mkdir immich-app
cd immich-app
~~~

~~~linux
wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
~~~

然后编辑.env文件,必须修改的只有三项:

**DB_PASSWORD,UPLOAD_LOCATION,TYPESENSE_API_KEY**

数据库密码,上传文件路径,和一个key,按照自己的情况填写,作者推荐key和数据库密码写成随机的

### 部署容器

然后就可以部署容器了,在immich-app文件夹下,输入

~~~linux
docker-compose up -d     # or `docker compose up -d` based on your docker-compose version
~~~

系统会自动部署.部署完成后显示如下

![immich over](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/immich over.jpg)

### 进入网站

然后进入 ip地址:2283,进入网站,输入你的邮箱和密码,就是管理员账号密码,之后可以设置普通用户.

下载immich github页面realese中的apk可以,输入ip地址:2283/api,可以连接到你的服务器,之后设置备份相册就可以了

<img src="https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/immich-andrioid login.png" alt="immich-andrioid login" style="zoom:25%;" />

给大家看看界面

![immich web界面](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/immich web界面.jpg)

<img src="https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/immich 安卓界面.png" alt="immich 安卓界面" style="zoom:25%;" />

## 后话

我的服务器配置2h4g 没跑分,估计就正常水平,线路是美西9929 100mbps,用起来偶有卡顿,可能因为我是校园网的原因,第一次上传照片(133张)直接给服务器干崩了,但是后面就好多了,上传的时候比较卡,当然也是我服务器网络和性能羸弱的原因.

想折腾的,有条件的搞一搞,没条件的老老实实网盘备份吧,不比这差
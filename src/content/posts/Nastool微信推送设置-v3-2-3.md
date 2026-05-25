---
title: "Nastool微信推送设置-V3.2.3"
published: 2023-07-29
tags:
  - nas
  - wechat
draft: false
toc: true
lang: zh
---
# Nastool微信推送设置-V3.2.3

昨天搞了一下nastool的微信推送,看了网上的攻略大部分都是爬的什么值得买的攻略(笑),本篇致力于写一篇详细的完整的小白的推送全设置,包括**frp设置**,**搭建微信代理服务器**

如果**只需要消息通知不需要交互**,那么是不需要Frp的.

## 条件准备

- 企业微信
- nas公网IP/Frp
- 一台有公网的服务器(做微信代理服务器)

## 创建企业微信

首先现在手机上下载企业微信.

网页搜企业微信,立即注册,信息按实填写就可以.![weixin1](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/weixin1.png)

使用自己的管理员微信扫描绑定后,就可以登陆到管理后台了,**使用手机企业微信登录进去**,同意邀请后就可以看到成员是1个人.![weixin2](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/weixin2.png)

点击上面菜单的**应用管理**,**自建-创建应用**,![weixin3](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/weixin3.png)

按实际填写就可以了.

填写完成后在应用管理界面就可以看到AgentId,Secret等信息了.

## Nastool设置

打开Nastool的消息通知,新建一个微信推送,可以看到填写的有以下几项:![weixin4](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/weixin4.png)

企业ID:企业微信-我的企业-企业ID

应用Secret:应用管理-Secret

应用ID: AgentId

## 应用设置

以上填完就可以用微信的消息推送了,但还不能用交互功能,需要填写下面三个.打开应用管理的接收消息

![image-20230729095957369](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230729095957369.png)

需要填写URL,Token,EncodingAESKey,后两个自动生成即可,**URL需要注意,需要你有公网IP或者是FRP**,如果你有公网IP或已经设置好FRP,请跳过下面的FRP设置.

现在Nastool机器人中将Token,EncodingAESKey输入其中并保存.(否则在企业微信输入URL后会连接不通)

输入你的**IP/域名:3000/wechat**(例:aprdec.top:3000/wechat),注意后面有**/wechat**,然后保存即可.

## FRP设置

如果没有公网IP,可以使用FRP来作映射,也是可以的,并且市面上有很多免费的FRP,而如果你只需要内网穿透用作微信通知,对服务端要求很小,**免费的足够**.如果你有需求也可以自己搭建FRP服务端,但本篇不再写如何搭建FRPS服务端.

我使用的是OpenFrp,还有其他很多免费的诸如sakura frp,大家的使用方法几乎都是一致的.

在OpenFrp创建账号后,点击创建隧道,选一个服务端.如果你不想实名,可以用国外的节点,都是可以的,我使用的日本节点,并没有什么影响

![image-20230729101742787](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230729101742787.png)

远程端口可以随机,提交即可.

### 安装软件

随后下载FRP软件,根据你的系统选择,我是linux,架构是x86,所以选择linux-amd64,如果不确定系统架构,使用

~~~
uname -a
~~~

选择对应架构的版本就可以了,更多的看[FRPC 使用教程]([FRPC 使用教程 | OpenFrp Docs](https://docs.openfrp.net/use/frpc.html#windows))

命令行连接nas/软路由,找一个路径下载程序.

譬如:wget /usr/local/bin/https://sq.oss.imzzh.cn/client/OpenFRP_0.49.0_5cc2e1cc_20230618/frpc_linux_amd64.tar.gz

也可以自己手动下载上传.

随后cd到下载程序的目录,使用

~~~bash
tar -zxvf <tar.gz文件>
~~~

解压压缩包,随后输入以下命令

~~~bash
chmod 755 <解压后出来文件名>
~~~

紧接着输入(./解压出来文件名)

~~~bash
./frpc_linux_amd64
~~~

就已经运行程序了.

会显示输入访问秘钥,去openfrp的首页就可以看到了,![image-20230729102845857](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230729102845857.png)粘贴一下,按tab来到下面continue,按回车.会显示你已经创建好的隧道,按下回车,在按tab,continue就可以了,此时会有连接的信息,**最后一行会给你域名和IP地址.记下域名,输入到企业微信回复消息的URL中就可以了.**

### 后台及开机自启

输入(以我自己的路径举例,使用时改成自己的路径)

~~~bash
nohup /usr/local/bin/frpc_linux_amd64 -u 访问秘钥 -p 隧道ID,隧道ID,.....(隧道ID可以是一个或多个)
~~~

开机自启,我是openwrt,有自带的开机运行命令,其他请自己网上搜开机命令

openwrt找到/etc/rc.local,在exit 0上面加入

~~~bash
/usr/local/bin/frpc_linux_amd64 -u 访问秘钥 -p 隧道ID,隧道ID,.....(隧道ID可以是一个或多个) &
~~~

就可以开机自启并且后台运行了

![image-20230729103700634](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230729103700634.png)

## 微信代理服务器

此时微信机器人需要填写的就只剩下**消息推送代理**了,我们需要自己找一台服务器来搭建,服务器需要安装nginx.

nginx安装教程不再赘述.

找到nginx的nginx.conf文件,如果是自己安装,一般在/etc/nginx/下,如果用宝塔安装,在/www/server/nginx/conf/下,

http{}中加入include /www/server/nginx/conf/sites-enabled/*;#路径根据自己的nginx配置文件位置修改

随后创建sites-enabled文件夹,在文件夹中创建wechat文件,文件中键入

~~~nginx
server
{
listen 5001; #修改⽆占⽤端⼝
location /cgi-bin/gettoken {
proxy_pass https://qyapi.weixin.qq.com;
}
location /cgi-bin/message/send
{
proxy_pass https://qyapi.weixin.qq.com;
}
}
~~~

随后

~~~bash
systemctl restart nginx
systemctl enable nginx(设置自启)
~~~

然后在nastool微信代理填入http://ip:5001就可以了,还需要在企业微信-应用管理-企业可信IP中填入该IP

![image-20230729105130709](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230729105130709.png)

完成到这里,你就可以完整体验微信交互功能了

## 小结

如果不需要交互,FRP设置一次后可以不在启动,只是不能交互了,推送消息还是可以的.

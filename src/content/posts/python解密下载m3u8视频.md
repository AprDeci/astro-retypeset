---
title: "python解密下载m3u8视频"
published: 2024-04-06
tags:
  - python
  - video
draft: false
toc: true
lang: zh
---
# python解密下载M3U8视频

寒假爬虫学习了一下,然后拖啊拖啊,就到了清明节了.

## m3u8文件内容

.m3u8文件内容其实就是一整个视频被切割成了N个.ts文件,通过逐个加载其中的ts文件来播放视频(肯定不准确,就这么理解吧)

> 与传统的视频格式不同，M3U8视频格式将整个视频**分成多个小片段进行传输**，这些小片段可以根据网络情况自动调节其质量和大小。这种方式使得M3U8视频格式非常适合在网络环境不稳定或带宽不足的情况下播放视频。
>
> 另外，M3U8视频格式还支持多种分辨率和比特率，以及字幕和音轨等多种附加信息。这些功能都使得M3U8视频格式成为了现代流媒体领域中的一种重要技术。
>
> 总的来说，M3U8视频格式具有高**效、灵活、可扩展**等优点，因此被越来越多的视频网站和应用所采用。如果您想深入了解M3U8视频格式，接下来我们将介绍如何解析M3U8视频地址，以及如何使用M3U8视频播放器播放这些视频文件。

每一个M3U8文件开头,都有如下几行![image-20240406213010175](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240406213010175.png)

**EXT-X-KEY**后面的信息即为M3U8加解密相关的内容.**METHOD**指**加密方式**(如图是AES-128),**URI**的内容即为该文件解密的**key文件地址**,**IV**为该文件**加密偏移量**

![image-20240406213314761](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240406213314761.png)

下面的就是各.ts文件的地址

## python代码

大致思路是 

> 获取m3u8文件->解析文件内容获取秘钥,偏移,加密方式,ts文件地址->循环获取解密ts文件->合并ts文件

爬取网络页面获取m3u8这里不再赘述

加密方式需要提前确定,可以自己先查看是什么加密方式,这里以AES-128为例

### 解析m3u8

使用正则解析获取key和iv

~~~python
# 获取key文件地址
key_url = re.findall('#EXT-X-KEY:METHOD=AES-128,URI="(.*)",IV=', m3u8_data)[0]
# 获取iv
iv = re.findall('#EXT-X-KEY:METHOD=AES-128,URI=".*",IV=(.*)', m3u8_data)[0][:16]
# 获取ts文件地址列表
ts_list = re.sub('#E.*', '', m3u8_data).split()
# 获取key文件内容
key = requests.get(key_url, headers=headers).content
~~~

### 解密合并ts

循环解密下载ts文件地址,并且通过file.write写入到同一个文件既合并

~~~
# 解码器 Crypto.Cipher
ci = AES.new(key, AES.MODE_CBC, iv=iv.encode())
current = current_process()
pos = current._identity[0] - 1
# 这里使用了tqdm显示进度
for ts in tqdm(ts_list, desc=name,position=pos):
# 加密数据
ts_content = requests.get(ts, headers=headers).content
# 解密数据
content = ci.decrypt(ts_content)
# 保存数据
with open(f'video\\{name}.mp4', 'ab') as f:
	f.write(content)
~~~


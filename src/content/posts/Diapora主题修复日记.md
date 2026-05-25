---
title: "Diapora主题修复日记"
published: 2024-02-17
tags:
  - wordpress
  - theme
draft: false
toc: true
lang: zh
---
# Diapora主题修复日记

我的环境nginx+PHP8.1+Wordpress6.4.3

很喜欢这个主题,但是年代有些久远,在wordpress6.4.3上有一些问题,于是决定修复+修改

## 字数统计

没有经过修改会提示变量没有初始化

![QQ截图20240216153138](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/QQ截图20240216153138.png)

查看源代码发现返回text如果为''会自动获取文章内容经过处理后以UTF-8编码计数![image-20240216153731740](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240216153731740.png)

![image-20240216153516021](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240216153516021.png)

改为

~~~php
<span class="icon-letter"><?php  echo count_words (''); ?></span>
~~~

出现如下情况,字体数量逐渐叠加

![QQ截图20240216153301](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/QQ截图20240216153301.png)

原因是查看字数函数会拼接$output,output没有刷新.查看函数竟然是因为我为了修复$post,$output没有初始化使用了global,改为

~~~php
function count_words ($text) {   
    global $post;  
	$output = '';
    if ( '' == $text ) {   
        $text = $post->post_content;  
		//$text = get_the_content(); 
        if (mb_strlen($output, 'UTF-8') < mb_strlen($text, 'UTF-8')) $output .= '' . mb_strlen(preg_replace('/\s/','',html_entity_decode(strip_tags($post->post_content))),'UTF-8') . '';   
        return $output;   
    }   
}
~~~

OK啦

![image-20240216154307439](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240216154307439.png)

## 随机特色图

主题本身不支持以API的方式传入随机文章缩略图,而是必须每个文章单独设置特色图或使用统一的默认图,对我这样的懒狗有些不友好

查找源代码(注释为原代码)![image-20240216154901363](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240216154901363.png)

![image-20240216154600840](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240216154600840.png)

修改img[0]路径即可,注册rest api保证路径统一

![image-20240216155036011](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240216155036011.png)

## 样式调整

首页大图存在拉伸缩放问题,调整为

~~~CSS
object-fit:contain;
~~~

### 缩略图

缩略图由于图片大小不同存在高度不一致问题,修改为

~~~CSS
object-fit:contain;
height:382.5px;
~~~

## 懒加载

Lazyload.js梭哈

## 后台

使用框架**codestarframework**,提醒一下option framework应该不兼容当前版本,不想自己修的就别用了.

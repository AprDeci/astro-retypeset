---
title: "vue3+vant4写堡垒之夜APP学习到的知识"
published: 2024-01-17
tags:
  - vue
  - vant
draft: false
toc: true
lang: zh
---
# Vue3+Vant4写堡垒之夜APP学习到的知识

`<van-nav-bar :title="nn" fixed placeholder/>`

文档中说fixed,placeholder接收boolean值,不论写'true'/true都会报错,实则不用写值,直接写参数名即可

## 学习的文章

[卡片布局自适应方案全解析_vue卡片布局-CSDN博客](https://blog.csdn.net/CooolYvonne/article/details/115254623)

[vue 中 v-for 动态生成盒子换行排列布局效果 - 掘金 (juejin.cn)](https://juejin.cn/post/7034318820718149668)

[🌟 CSS 幻术 | 有关光影效果的黑魔法 - 掘金 (juejin.cn)](https://juejin.cn/post/6965488051695353886)

[如何用纯 CSS 创作一个金属光泽 3D 按钮特效_纯css3实现开机按钮 按钮3d有光泽-CSDN博客](https://blog.csdn.net/w178191520/article/details/84260915)

[FastApi使用定时任务_fastapi 定时任务-CSDN博客](https://blog.csdn.net/qq_51967017/article/details/131058627)

[Ubuntu部署FastApi项目 - 莫问哥哥 - 博客园 (cnblogs.com)](https://www.cnblogs.com/LearningC/p/17336452.html)

[venv - 廖雪峰的官方网站 (liaoxuefeng.com)](https://www.liaoxuefeng.com/wiki/1016959663602400/1019273143120480)

[Let’s Encrypt SSL 证书的申请与使用_letsencrypt证书申请-CSDN博客](https://blog.csdn.net/doushi/article/details/128762885)

[Certbot Instructions | Certbot (eff.org)](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)

[奇妙的 CSS MASK · Issue #80 · chokcoco/iCSS (github.com)](https://github.com/chokcoco/iCSS/issues/80)

## Cloudflare Workers图片反代

因为商城是爬取的Tracker.fortnite的数据,图片链接也是trackercdn的,导致国内连接比较慢,本来在想要不要用后端爬虫自动存储到服务器中,用服务器传输图片,但终究用到自己服务器的流量,还是双向,真是心疼.所幸在Cloudflare page部署测试页面时候,看到了Workers,并且了解到Workers可以反代资源,于是使用Cloudflare Workers进行反代图片

代码复制的别的博主的

~~~javascript
addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
    const imageUrl = new URL(request.url);
    imageUrl.hostname = "源站域名";  // 替换成源站域名
    imageUrl.protocol = "https";  // 源站的http协议

    // 构建新的请求对象
    const imageRequest = new Request(imageUrl, request)

    // 删除 Referer 请求头，也可以使用 imageRequest.headers.set('Referer', '') 修改Referer 
    imageRequest.headers.delete('Referer')

    // 从远程服务器获取响应
    const imageResponse = await fetch(imageRequest)

    // 将响应体作为字节数组读取
    const imageBuffer = await imageResponse.arrayBuffer()

    // 构建新的响应对象
    const response = new Response(imageBuffer, imageResponse)

    // 设置 Content-Type 标头
    response.headers.set('Content-Type', imageResponse.headers.get('Content-Type'))

    // 返回响应对象
    return response
}
~~~



## CSS上的问题

### 文字根据容器大小自动调整Fontsize

~~~Vue
<script setup>
const namedom = ref()
function changefontsize(){
     if(containerdom.value.scrollWidth>containerdom.value.offsetWidth){
         namedom.value.style.fontSize = "10px"
         console.log("dadaddadada")
     }
}
onMounted(() => {
    changefontsize()
    console.log(containerdom.value.scrollWidth)
})
</script>
<template>
<div class="shop-card">
        <img class="item-img" v-lazy=imurl>
        <div class="item-info-container" ref="containerdom">
        <p class="item-name" ref="namedom">Festival bundle bundle bundle</p>
        <p class="item-price"><img style="width: 20px; vertical-align: middle;" src="@/assets/imgs/vbuck.png" alt="">1200</p>
    </div>
    </div>
</template>

<style>
    .item-name{

    color: white;
    letter-spacing: 2px;
    position: relative;
    left: 10px;
}
.item-price{
    font-size: 25px;
    color: white;
    letter-spacing: 2px;
    position: relative;
    left: 10px;
}
.item-info-container{
    position: relative;
    bottom: 110px;
    padding-right: 1px;
    max-width: 200px;
    height: 100%;
    background:#00000055;
    white-space: nowrap;//使文字不折叠
    overflow-x: auto; //获取scroll
}
</style>
~~~


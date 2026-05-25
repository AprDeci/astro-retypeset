---
title: "写一个valorant-storefront-bot推送tg机器人"
published: 2023-07-19
tags:
  - others
draft: false
toc: true
lang: zh
---
# 写一个valorant-storefront-bot 推送tg机器人

每天手动查每日商店有点累,有点烦,游民时不时抽个风都让我觉得难受,顺便也想学习一下node.js,所以就学了一下nodejs,尝试写了一个机器人来.

## loading.

valorant的api比较丰富,而且有大佬写的现成的库,实现起来十分简单(不论是tg机器人还是valorantapi的库)

只是不知道有没有办法将图片和文字放在一条消息里,看看tgbotapi也没找到相关的.

目前只实现了**每日商店自动推送和手动查询**,之后预计实现战绩查询和tg账号绑定valorant账号和环境变量,给同学用用.

![image-20230718190054308](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230718190054308.png)

## Webhook

tgbot接受消息有两种模式:**polling和webhook**,简单分就是polling简单但是耗费资源,webhook比较难不耗费资源.

实现方式.

~~~js
const express = require('express');
const bodyParser = require('body-parser');
const tgbot = require('node-telegram-bot-api');
const tgbottoken = 'Token'
const bot = new tgbot(tgbottoken,{polling:true})
const url = 'https://..'
const port = xxxxx 
 bot.setWebHook(`${url}/bot${tgbottoken}`)
  const app = express();
  app.use(bodyParser.json());
  app.get('/', (req, res) => res.send('Valorant-storefront-bot'));
  app.post(`/bot${tgbottoken}`, (req, res) => {
  bot.processUpdate(req.body);
  res.sendStatus(200);
});
  app.listen(port, () => {
    console.log(`Express server is listening on ${port}`);
});
~~~

接着在nginx映射端口.(待写)



## 备忘录

```
ps -ef | grep 进程关键字(node)
kill -9 pid
```

ctrl+c退出node命令行模式


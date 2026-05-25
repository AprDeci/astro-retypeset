---
title: "UmamiVercelSupabase免费搭建网站数据统计超详细"
published: 2022-12-29
tags:
  - github
  - umami
draft: false
toc: true
lang: zh
---
# Umami+Vercel+Supabase  免费搭建网站数据统计(超详细)

## 开端

**发现这个项目看到很多博主都发了免费搭建的教程,但是很多都是一年前的,随着Umami的更新,很多方法已经不适用,所以写篇详细的文章来演示一下搭建过程以及过程中碰到的问题和解决办法.**

效果见:[Umami演示]([umami (aprdec.top)](https://ver.aprdec.top/share/zBMTnKJE/孟夏十二の天空))

## 教程

### 一·数据库

首先进入[Supabase](https://app.supabase.com/)官网,并且注册账号登录,你也可以用Github直接登陆.

登录完成后点击New Project 创建一个数据库![supabase1](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase1.jpg)

填写数据库的名称,数据库密码,以及数据库所在地区,其他就不用动了<img src="https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase2.jpg" alt="supabase2" style="zoom: 67%;" />

创建好数据库后,可以看到左边Organizations以及NEW project的下面已经有了刚刚创建的数据库实例.点击进入管理界面,点击左边的SQL Editor

![supabase3](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase3.jpg)

点击进入Umami[**schema.postgresql.sql**]([umami/schema.postgresql.sql at master · umami-software/umami (github.com)](https://github.com/umami-software/umami/blob/master/sql/schema.postgresql.sql))文件,点击复制文件内容

![supabase4](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase4.jpg)

复制后粘贴到SQL Editor中,这时候注意,在v1.37.0后,有两个表名有改变,所以我们需要将

```sql
    "event_type" VARCHAR(50) NOT NULL,
    "event_value" VARCHAR(50) NOT NULL,
```

改成

```sql
"event_name" VARCHAR(50) NOT NULL,
"event_data" VARCHAR(50) NOT NULL,
```

然后点击右边的RUN,

![supabase5](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase5.jpg)

之后数据库便会进行初始化,并提示"Success. Now rows returned",然后我们返回查看表Table Editor便可以看到数据库已经有了创建的表**(最初是五个,这是我已经使用的)**![supabase6](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase6.jpg)

### 在1.3.9之后,Prisma迁移中会提示权限不足

我们需要在SQL_Editor中,复制以下代码点击RUN(方法均来自

[Github_Discussions#1542](https://github.com/umami-software/umami/discussions/1542)

)

```sql
ALTER TABLE account OWNER TO postgres;
ALTER TABLE website OWNER TO postgres;
ALTER TABLE session OWNER TO postgres;
ALTER TABLE pageview OWNER TO postgres;
ALTER TABLE event OWNER TO postgres;
ALTER TABLE event_data OWNER TO postgres;
```

然后还要解决Error:P3009问题,

如果电脑**有Node.Js环境,(没有看下面)**可以先把[Umami]([umami-software/umami: Umami is a simple, fast, privacy-focused alternative to Google Analytics. (github.com)](https://github.com/umami-software/umami))下载到本地,进行如下操作

- 在本地文件中创建一个.env文件,并且在.env文件中添加如下内容

```md
DATABASE_URL=postgres://postgres:[YOUR-PASSWORD]@[HOST]:6543/postgres?pgbouncer=true
HASH_SALT=any-random-string
```

​    这个链接可以在数据库的Setting里查看![supabase8](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase8.jpg)

- 然后在本地运行以下代码

  ```
  yarn install
  yarn prisma migrate resolve --applied "02_add_event_data"
  yarn build
  ```

  

**如果电脑没有NodeJS环境,可以使用GithubSpace功能**

- 首先Fork一下Umami库到自己的Github库里

![supabase9](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase9.jpg)

- 然后进入自己Fork的库中,点击Code,选择Codespace,创建一个新的space,![supabase10](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase10.jpg)

进入Codespace中是一个类似Vscode的界面,等待终端安装Nodejs,然后在按照上面的操作重复即可

![supabase11](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/supabase11.png)

### Vercel

如果之前没有fork作者仓库,我们需要fork到自己的github下

完成上面的操作,我们进入[Vercel](vercel.com)登陆/注册账号,并且绑定你的Github账号,然后选择Add New->Project,然后选择你Fork的Umami仓库

![Vercel1](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/Vercel1.jpg)

点击Import,Project Name填写你想要的,其他不用修改,点击Environment Variables添加环境变量**DATABASE_URL** 和**HASH_SALT**,DATABASE_URL和上面.env的URL一致,HASH_SALT可随意填写,也可以在这个网站生成[HASH_SALT]([MD5 Hash Generator](https://www.md5hashgenerator.com/))

![Vercel2](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/Vercel2.jpg)

然后点击Deploy,如果上面的步骤没有做错,应该就可以部署成功了.

等待部署成功后,我们进入Vercel给我们的二级域名(或者你自己绑定的域名,推荐自己绑定)

进入Umami的登陆界面,初始用户名 admin 密码 umami

![Umami1](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/Umami1.jpg)

登陆后进入设置可以更改密码,语言,时区等等.

然后在网站栏添加自己的域名,跟随指引引入Umami即可

---
title: "Sakura主题修改WordPress加速"
published: 2024-09-12
tags:
  - github
  - wordpress
  - theme
draft: false
toc: true
lang: zh
---
# Sakura 主题修改(Wordpress 加速)

sakura 主题的好是不言而喻的, 许多二次元的选择.我的博客也一直在使用并且持续的修改, 我也十分关注网站的速度, 网站从最初特别特别慢, 到现在逐渐变快, 是我一直在不断学习相关知识, 从最初的不懂到如今逐渐理解.但任重而道远, 我会在这篇博文记录我的一些优化博客速度的方法(防止我忘了)

我的博客的服务器是宽带 3M 的大陆服务器, 不知大伙访问我的博客速度如何?如果有朋友看到了这篇博客, 请留言告诉我你的体验, 谢谢!

## 滑动顺滑

使用了 [lenis: How smooth scroll should be (github.com)](https://github.com/darkroomengineering/lenis?utm_source=gold_browser_extension) 库.

在 header 或 footer 某个文件引入以下代码即可, 更多参数可以看官方文档

~~~
const lenis = new Lenis()

lenis.on('scroll', (e) => {
console.log(e)
})

function raf(time) {
lenis.raf(time)
requestAnimationFrame(raf)
}

requestAnimationFrame(raf)
~~~



## 暗黑模式扩散动画

修改文件 ![image-20240816171339812](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240816171339812.png)

**363 行 mobile_dark_light 修改如下**

~~~javascript
function mobile_dark_light(e) {
    var x = event.changedTouches ? event.changedTouches [0].clientX : (event.clientX || event.pageX);
    var y = event.changedTouches ? event.changedTouches [0].clientY : (event.clientY || event.pageY);
    console.log(x, y)
    const endRadius = Math.hypot(
        Math.max(x, innerWidth - x),
        Math.max(y, innerHeight - y),
      )
    document.documentElement.style.setProperty('--x', x + 'px')
    document.documentElement.style.setProperty('--y', y + 'px')
    document.documentElement.style.setProperty('--r', endRadius + 'px')
    if ($(" body ").hasClass(" dark ")) {
        if(document.startViewTransition) {
            document.startViewTransition(() => {
                $(" html ").removeClass(" dark ");
                $(" html ").css(" background ", " unset ");
                $(" body ").removeClass(" dark ");
                $("#moblieDarkLight ").html('< i class =" fa fa-moon-o " aria-hidden =" true "></i >');
                setCookie("dark", "0", 0.33);
            });
        }else {
            $(" html ").removeClass(" dark ");
            $(" html ").css(" background ", " unset ");
            $(" body ").removeClass(" dark ");
            $("#moblieDarkLight ").html('< i class =" fa fa-moon-o " aria-hidden =" true "></i >');
            setCookie("dark", "0", 0.33);
        }

    } else {
        if(document.startViewTransition){
            document.startViewTransition(() => {
                $(" html ").addClass(" dark ");
                $(" html ").css(" background ", "#31363b ");
                $("#moblieDarkLight ").html('< i class =" fa fa-sun-o " aria-hidden =" true "></i >');
                $(" body ").addClass(" dark ");
                setCookie("dark", "1", 0.33);
            })
            }else{
                $(" html ").addClass(" dark ");
                $(" html ").css(" background ", "#31363b ");
                $("#moblieDarkLight ").html('< i class =" fa fa-sun-o " aria-hidden =" true "></i >');
                $(" body ").addClass(" dark ");
                setCookie("dark", "1", 0.33);
            }
        }

    }
~~~

**395 行 changeBG()修改如下**

~~~javascript
function changeBG() {
        var cached = $(".menu-list ");
        cached.find("li").each(function () {
            var tagid = this.id;
            cached.on("click", "#" + tagid, function (e) {
                if (tagid == "white-bg" || tagid == "dark-bg") {
                    mashiro_global.variables.skinSecter = true;
                    checkskinSecter();
                } else {
                    mashiro_global.variables.skinSecter = false;
                    checkskinSecter();
                }
                if (tagid == "dark-bg") {
                    addComment.I("content").classList.add('notransition');
                    addComment.I("content").style.backgroundColor = "#fff";
                    addComment.I("content").offsetHeight;
                    addComment.I("content").classList.remove('notransition');
                    const endRadius = Math.hypot(
                        Math.max(e.clientX, innerWidth - e.clientX),
                        Math.max(e.clientY, innerHeight - e.clientY),
                      )
                    document.documentElement.style.setProperty('--x', e.clientX + 'px')
                    document.documentElement.style.setProperty('--y', e.clientY + 'px')
                    document.documentElement.style.setProperty('--r', endRadius + 'px')
                    if(document.startViewTransition){
                        document.startViewTransition(() => {
                            $(" html ").addClass(" dark ");
                            $(" html ").css(" background ", "#31363b ");
                            $(" body ").addClass(" dark ");
                            setCookie("dark", "1", 0.33);
                        })
                    }else{
                        $(" html ").css(" background ", "#31363b ");
                        $(" body ").addClass(" dark ");
                        setCookie("dark", "1", 0.33);
                    }

                } else{
                    if(document.startViewTransition){
                        document.startViewTransition(() =>{
                            $(" html ").removeClass(" dark ");
                            $(" html ").css(" background ", " unset ");
                            $(" body ").removeClass(" dark ");
                            setCookie("dark", "0", 0.33);
                            setCookie("bgImgSetting", tagid, 30);
                            setTimeout(function () {
                                addComment.I("content").style.backgroundColor = "rgba(255, 255, 255, 0.8)";
                            }, 1000);
                        })
                    }else{
                        $(" html ").css(" background ", " unset ");
                        $(" body ").removeClass(" dark ");
                        setCookie("dark", "0", 0.33);
                        setCookie("bgImgSetting", tagid, 30);
                        setTimeout(function () {
                            addComment.I("content").style.backgroundColor = "rgba(255, 255, 255, 0.8)";
                        }, 1000);
                    }

                }
                switch (tagid) {
                    case "white-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg0) + ")");
                        break;
                    case "sakura-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg1) + ")");
                        break;
                    case "gribs-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg2) + ")");
                        break;
                    case "pixiv-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg3) + ")");
                        break;
                    case "KAdots-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg4) + ")");
                        break;
                    case "totem-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg5) + ")");
                        break;
                    case "bing-bg":
                        $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg6) + ")");
                        break;
                    // case "dark-bg":
                    //     $(" body ").css(" background-image ", " url(" + checkskin_bg(mashiro_option.skin_bg7) + ")");
                    //     break;
                }
                closeSkinMenu();
            });
        });
    }
~~~

**style.css 添加如下代码**

~~~css
  @keyframes clip {
    from {
      clip-path: circle(0% at var(--x) var(--y));
    }
    to{
      clip-path: circle(var(--r) at var(--x) var(--y));
    }
  }                 
  :: view-transition-old(root) {
    animation: none;
  }

  :: view-transition-new(root) {
    animation: clip 0.5s ease-in;
  }

  html.dark:: view-transition-old(root) {
    animation: clip 0.5s ease-in reverse;
  }

  html.dark:: view-transition-new(root) {
    animation: none;
  }

  html.dark:: view-transition-old(root) {
    z-index: 9999;
  }

  html.dark:: view-transition-new(root) {
    z-index: 1;
  }
~~~



## 特色图添加后缀

七牛云 cdn 转换 webp 需要在图片链接添加转换后缀, 修改文件如下

![image-20240814112339595](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240814112339595.png)

两处修改 ![image-20240814112420600](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240814112420600.png)

## 文章设置特色图后文章页头部依旧是随机图

有些文章设置了特色图, 但文章页面头图依旧是随机图, 希望头图是特色图

修改文件如下

![image-20240814122435332](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240814122435332.png)

将图片中该行注释(该行作用为: 如果设置了随机图则优先使用随机图)

![image-20240814122559356](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240814122559356.png)

如要添加后缀如下

![image-20240814123343817](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240814123343817.png)

## 默认启用 wordpress 自带 lightbox

wordpress6.4 更新了图片灯箱, 在主题文件夹创建 theme.json,

输入如下开启默认启用 lightbox

~~~
{
	"version": 2,
	"settings": {
		"blocks": {
			"core/image": {
				"lightbox": {
					"enabled": true
				}
			}
		}
	}
}
~~~

## 一些修改过程中主题因为 php 版本的报错

后台报错: Automatic conversion of false to array is deprecated in C:\Users\p\Local Sites\sakuradev\app\public\wp-content\themes\sakura-3.4.0\inc\categories-images.php on line 11

修改如下

![image-20240816130102969](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20240816130102969.png)

### Deprecated: substr(): Passing null to parameter #1 ($string) of type string is deprecated in C:\Users\p\Local Sites\sakuradev\app\public\wp-content\themes\sakura-3.4.0\tpl\content-thumb.php on line 48

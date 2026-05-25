---
title: "python写动物派对挂机刷经验脚本"
published: 2023-10-05
tags:
  - python
  - game-script
draft: false
toc: true
lang: zh
---
[toc]

## 每日闲言

yolov5为什么一直警告我软件包版本不对!

## 为何想做

动物派对发售了,虽然一致决定不买了,但还是忍不住在测试的最后两天买来玩了玩,当时不知道正式版后升级奖励给不给了,就想写个挂机脚本刷经验,需求也很简单,**帮我自动匹配,然后在游戏里瞎蹦跶两下**就好了,而且也一直想尝试做脚本,正好这次学习一下

## 难点

在查询资料之后,决定使用pyautogui来模拟我的操作,这个库模拟键盘鼠标的操作还是很方便的,使用起来很方便,对我第一次写脚本而言,让我头疼的是整个脚本的逻辑,我选择了**图片识别**来获取坐标点,来达成自动匹配确定英雄的操作.

在初次尝试写出代码后,感到最难办的还是逻辑问题,只简单地写出while循环,持续的检测并不难办,但是还要考虑做出一个操作后有可能这个操作就不需要了,有可能某个操作在两个操作之间才会执行.如果只是无脑的对所有图像进行检测,效率则会降低很多.

写到这里,不想在长篇大论了.对我而言感受到的难点有如下

- pyautogui的识图精度低下
- 不同操作之间的相互关系实现
- 游戏内模拟操作

## 编写历程

对于pyautogui的识图精度,真的是一言难尽,confidence调低了瞎匹配,调高了不匹配.在经历了几次改版后,换用了opencv2的matchTemplate算法,通过设定阈值,精度可以达到比较完美的效果,这里是看了B站一个UP主的视频了解到.

~~~python
threshold = 0.02#设定阈值
def getxy(img_path):
    img = cv2.imread("./imgs/screenshot.png")
    img_aim = cv2.imread(img_path)
    height,width,channel = img_aim.shape
    result = cv2.matchTemplate(img,img_aim,cv2.TM_SQDIFF_NORMED)
    min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)
    if min_val <threshold:
        upper_left = min_loc
        # 计算出匹配区域右下角图标（左上角坐标加上模板的长宽即可得到）
        lower_right = (upper_left[0] + width, upper_left[1] + height)
        # 计算坐标的平均值并将其返回
        avg = (int((upper_left[0] + lower_right[0]) / 2), int((upper_left[1] + lower_right[1]) / 2))
    else:#左上角坐标
        avg = None
    return avg
~~~



对于不同操作的关系,譬如结束游戏一定在开始游戏之后,游戏模拟操作一定是在准备之后结束游戏之前,但我python技术一般,不会太高大上的操作,只能用变量作为依据控制操作与否,所以换了思路,为每个识图操作设定时间戳,在执行一次后刷新时间戳,直到界定时间才再次检测

~~~python
last_executed_times = {
    'begin_room.png': 0,
    'begin.png': 0,
    'ready.png': 0,
    'endgame.png': 0,
    'expup2.png': 0
}
def execute_condition(condition, last_executed_times, interval):
    if current_time - last_executed_times[condition] >= interval:
        last_executed_times[condition] = current_time
        return True
    else:
        return False
while True:
        if execute_condition('ready.png',last_executed_times, 50):
           ...
~~~



还是用变量实现模拟操作在ready后end前.并且每一次识图后,如果检测到则不再执行action()

游戏里的模拟操作,就用pyautogui的keydown和keyup来执行了.

目前写下来,问题还是有不少,但是基本能平稳使用了.目前而言最大的问题是时间戳的问题,是要在图像检测前就判定时间戳,还是要每次都检测图形再判定时间戳,无疑后者性能差点,但是速度也是后者更快,不知道有什么更好的解决办法.

最后,八戒镇楼

![image-20230923160021366](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230923160021366.png)
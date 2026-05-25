---
title: "动物派对挂机脚本学习记录(Pyqt5)"
published: 2023-10-05
tags:
  - python
  - game-script
draft: false
toc: true
lang: zh
---
# 动物派对挂机脚本学习记录(Pyqt5)

编写动物派对的脚本过程中,让我对python的使用有了巨大的进步,当然不只是因为写脚本,这一段时间一直在看关于python的东西,这会再写一篇文章来记录九月我的收获.

上一篇写完的时候我的脚本还只是一个简陋的py文件,功能不完善脚本也存在一定的差错问题.这星期抽时间学习了Pyqt5,给动物派对挂机脚本做了UI界面,并且添加了**自建房间**功能,其他功能也如图片所见,可以说像是一个比较成熟的脚本了.

在给脚本写UI界面的过程中,我学习了很多,于是决定单独开篇文章来记录,当然,代码部分我也没有放下,有什么想到的我都会在下面写.

![动物派对GUI](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/动物派对GUI.gif)

## 需求

我先来写下该GUI需要实现的功能,以免我忘了

- 执行脚本不卡死
- 实现限定局数和挂机后关机的功能
- 两个按钮的联动(挂机后关机点击同时开启限定局数)
- 实现中止功能

## Qthread类的使用

其实最初界面并不是这样,是一个十分简陋的界面(学了半天同学要用赶紧写了个UI),当时还想把一些参数给他们让他们自定义,后来发现不太需要.

当然问题也很明显,丑陋我们先略过不表,最重要的是我的脚本是一个无限循环函数,每次点击开始脚本GUI都会**无响应**,搜索了一下得知这是耗时函数的通病,解决办法就是用多线程的方式另开一个线程来执行主函数.

![image-20230929171451896](https://luchetuchuang.oss-cn-beijing.aliyuncs.com/aprimg/image-20230929171451896.png)

在多方(AI)帮助下,我在主文件中新建了类WorkThread继承QThread.因为在执行主函数时候需要传参(方便设置局数和关机),所以在初始化时同时创建三个参数.重写run函数,因为不同按钮信号不同,所以也要考虑**WorkThread的通用性**,最终代码如下:(stop方法后面讲)

~~~python
class WorkerThread(QThread):
    finished = pyqtSignal()  # 操作完成时发射的信号
    def __init__(self, operation_func, *args, **kwargs):
        super().__init__()
        self.operation_func = operation_func
        self.args = args
        self.kwargs = kwargs
    def run(self):
        self.operation_func(*self.args, **self.kwargs)
    def stop(self):
        self.terminate()
        self.finished.emit()
class MymainWindow(QMainWindow, Ui_mainWindow)#窗口类:
		init(self):
            ```
            原代码

            ```		
            self.operation_thread = None ### 初始化workthread
        def start_operation_thread(self, operation_func, *args, **kwargs):
        if not self.operation_thread or not self.operation_thread.isRunning():
            self.operation_thread = WorkerThread(operation_func, *args, **kwargs)
            self.operation_thread.finished.connect(self.on_operation_finished)
            self.operation_thread.start()
            match operation_func:
                case room.roomgame:
                    self.roombuttom.setText("停止脚本")
                    self.quickgamebutton.setEnabled(False)
                case main.main:
                    self.quickgamebutton.setText("停止脚本")
                    self.roombuttom.setEnabled(False)
        else:
            self.operation_thread.stop()
            match operation_func:
                case room.roomgame:
                    self.roombuttom.setText("自建房间")
                    self.quickgamebutton.setEnabled(True)
                case main.main:
                    self.quickgamebutton.setText("快速游戏")
                    self.roombuttom.setEnabled(True)
        def on_operation_finished(self):
            pass
        def beginquick(self):
            self.getinfo()
            self.start_operation_thread(main .main, self.limitNumber, self.shutdown, self.gameNumber)
        def beginroom(self):
            self.start_operation_thread(room.roomgame)
~~~

## 限定局数和关机功能

这个其实没啥讲的,需要看的应该也是小白,我的做法是在一局结束后的某个节点判断是否点击了限定局数，如果点击则判断当前局数和目标局数，写了一个函数来判断是关闭游戏还是关机

~~~
#伪代码
def quit（isexit）：
	if isexit
		关机
	else
		退出游戏
if 限定局数=True：
	if current = goal：
		quit（isexit）
		
~~~

## 两个按钮联动

因为使用QtDesigner，所以选择继承类来运行窗口，init方法添加如下，使用lambda

~~~python
self.ifshutdown.clicked.connect(lambda: self.ifNumber.setChecked(True))
~~~

## 终止功能

因为我的main方法是while true形式的无限循环，所以无法在进程类插入stop flag实现停止，只能强制终止进程。网上搜了搜quit（）和wait（）方法都不好用，会导致GUI卡死，**如果有大佬知道怎么使用quit还不无响应，请一定告诉我**，所以只能使用.terminate()方法强制关闭线程.

[^terminate()]: 暴力结束的线程可能会用到互斥锁，使用QMutexLocker方法进行线程互斥，如果暴力结束，栈对象好像没有释放，导致QMutexLocker没有执行析构函数，导致没有解锁，产生死锁；(CSDN复制的)

~~~python
class WorkerThread(QThread):
    def stop(self):
        self.terminate()
        self.finished.emit()
class MymainWindow(QMainWindow, Ui_mainWindow):
 def start_operation_thread(self, operation_func, *args, **kwargs):
        if not self.operation_thread or not self.operation_thread.isRunning():
            self.operation_thread = WorkerThread(operation_func, *args, **kwargs)
            self.operation_thread.finished.connect(self.on_operation_finished)
            self.operation_thread.start()
            match operation_func:
                case room.roomgame:
                    self.roombuttom.setText("停止脚本")
                    self.quickgamebutton.setEnabled(False)
                case main.main:
                    self.quickgamebutton.setText("停止脚本")
                    self.roombuttom.setEnabled(False)
        else:
            self.operation_thread.stop()
            match operation_func:
                case room.roomgame:
                    self.roombuttom.setText("自建房间")
                    self.quickgamebutton.setEnabled(True)
                case main.main:
                    self.quickgamebutton.setText("快速游戏")
                    self.roombuttom.setEnabled(True)

~~~

先写这么多吧,感冒了(2023.9.29),另外,使用了pyfluentUI组件库,大佬很强,库很好用,后期会美化


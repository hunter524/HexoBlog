---
title: Handler Looper Message MessageQueue源码分析及API说明
tags: Android常用类源码分析
date: 2017/2/8 19:05:26
---

这是笔者的第一篇Android相关技术的博客，首先提供一下基本学习资料获取方式。如果是只需要了解API的基本使用只需要下载Android SDK完整版即可，里面包含了我们日常使用的SDK的源码，看这部分源码通常只需要在Android Studio里面进行简单的配置便可以阅读。同时由于使用的是比较熟悉的开发工具进行阅读代码当然效率也就更高了。但是有一部分FrameWork的代码通常很难使用Android Studio进行索引和查找，此时我们便要单独去下载源码使用其他源码阅读工具进行阅读。[Android FrameWork 源码下载](https://github.com/android/platform_frameworks_base),笔者使用的源码阅读软件为UnderStand4.0当然也可以使用SourceInsight等软件。

# 从Handler最简单的使用说起  
## Handler用于子线程异步执行耗时操作，执行完毕刷新界面
Android开发过程当中最基本的一个原则是不能在主线程中进行耗时操作。主线程通常我们是指ActivityThread，这边对于ActivityThread不详细展开。我们先暂且理解为这个线程启动了就代表应用启动了，这个线程结束了就代表应用结束了。如界面绘制，响应用户的点击操作等都是在主线程进行的。一个线程在同一时刻只能执行一个操作，如果在主线程中执行了耗时操作便会导致后续的操作被延时执行。如果延迟了界面绘制操作给用户的交互感觉便是界面卡顿不流畅，如果延迟对用户点击操作的响应用户便会感觉点击某个按钮始终得不到响应。
所以我们通常的做法便是在Activity或者Fragment内部new一个Handler对象，然后再构造一个子线程执行耗时操作，执行结束之后再用之前new的Handler对象post一个Runnalble执行界面的刷新操作，代码如下：
````  
    Handler mMainHandler = new Handler();

    Thread mWorkThread = new Thread(new Runnable() {
        @Override
        public void run() {
          //执行耗时操作
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
          //耗时操作执行结束，刷新页面
            mMainHandler.post(new Runnable() {
                @Override
                public void run() {
                    mNormalBt.setText("任务处理结束！");
                }
            });
        }
    });

    mWorkThread.start();
````
这部分代码分为三个流程：
1、new 一个Handler对象。
2、new 一个Thread，然后start执行耗时操作。
3、耗时操作执行完成之后通过之前new Handler对象post 一个Runnable对象执行界面更新操作。  

第一个流程new Handler对象，我们没做线程切换操作，当然是在主线程中执行了对象的创建操作，在主线程中执行Handler对象构建操作这一点在后面的分析中是一个重点。第二个流程是new 一个Thread对象，这个Thread对象的构造和之前Handler对象的构造同样在主线程中执行，对象构造的过程我们认为是几乎不耗费时间的。然后我们执行了mWorkThread.start()。至此主线程已经结束了它的的所有工作，主线程便结束了它的所有工作，主线程可以继续执行界面刷新，响应用户的点击等操作。

在子线程中我们为了模仿耗时操作让子线程sleep了5秒，然后使用我们在主线程中构造的mMainHandler.post()发出了一个Runnalbe，在Runnable中执行了界面的刷新操作，让按钮显示 “任务处理结束！”。这个执行界面刷新的Runnable其实是在主线程中得到调用和执行的。

至此我们可能会产生一个大大的疑问：为什么mMainHandler.post()发出的Runnalbe是在主线程中执行的，而不是在子线程中执行的？我明明是在子线程中去post这个Runnable的啊！这里是怎么做线程切换的？
## 基于源码的流程分析




runWithScissor方法阻塞调用线程直到当前事件处理结束

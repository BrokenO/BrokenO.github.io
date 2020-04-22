---
title: Activity的生命周期
tags:
  - blog
categories:
  - Android
toc: false
date: 2020-04-19 23:37:41
---

### 借用神图一张
![image.png](/images/2020/04/19/fbb42a90-824e-11ea-92f8-ab273354db60.png)

### 生命周期职责

1.attach ：严格意义上来说attach算不上是activity生命周期的一部分。当activity被反射创建之后，会调用该方法对activity进行初始化，此时会做下面几个主要的工作：
- 创建PhoneWindow对象
- 为PhoneWindow设置WindowManager
- 初始化Fragment控制器

2.onCreate ：主要用于控件的设置与初始化，最主要的是viewTree的构建，功能如下：
- 通过setContentView解析布局文件，从而构建ViewTree，并对其进行初始化
- 该方法在activity整个生命周期中只会调用一次
- 如果当前的activity是由于内存不足等原因被异常结束再回复，则该函数中的参数将不会为空。

3.onStart：此方法被回调时表示Activity正在启动，此时Activity已处于可见状态，只是还没有在前台显示，因此无法与用户进行交互。可以简单理解为Activity已显示而我们无法看见摆了。如果当前activity是回收后被恢复，则在改方法或会执行onReatoreInstanceState方法

4.onResume：当此方法回调时，则说明Activity已在前台可见，可与用户交互了，onResume方法与onStart的相同点是两者都表示Activity可见，只不过onStart回调时Activity还是处于后台无法与用户交互，而onResume则已显示在前台，可与用户交互。此方法主要的工作：
- 则获取attach中的PhoneWindow，将window添加到WindowManagerService中去
- 通过viewRootImp的添加开始DecorView的三大流程

5.onPause：

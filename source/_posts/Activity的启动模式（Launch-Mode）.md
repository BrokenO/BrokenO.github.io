---
title: Activity的启动模式（Launch Mode）
tags:
  - blog
categories:
  - Android
toc: false
date: 2020-04-19 16:12:44
---

### Activity启动模式种类：
![image.png](/images/2020/04/19/43fefa20-8211-11ea-92f8-ab273354db60.png)

![image.png](/images/2020/04/19/00926dc0-8212-11ea-92f8-ab273354db60.png)

activity可以添加**taskAffinity**选项，用来指定当前activity存放的任务栈，默认情况下该选项是当前应用的包名，**此选项只有在singleTask模式下才有效果**。如果当前存在一个Task栈，且其taskAffinity与当前activity不同，则系统会生成另一个Task栈用来存放当前的activity


#### Standard 常规模式（系统默认）

  这个模式是默认的启动模式，即标准模式，在不指定启动模式的前提下，系统默认使用该模式启动Activity，每次启动一个Activity都会重写创建一个新的实例，不管这个实例存不存在，这种模式下，谁启动了该模式的Activity，该Activity就属于启动它的Activity的任务栈中。这个Activity它的onCreate()，onStart()，onResume()方法都会被调用。

#### SingleTop（栈顶复用）

1.当前栈中已有**该Activity的实例并且该实例位于栈顶时**，不会新建实例，而是复用栈顶的实例，并且会将Intent对象传入，回调**onNewIntent**方法

2.当前栈中已有该Activity的实例但是该实例不在栈顶时，其行为和standard启动模式一样，依然会创建一个新的实例

3.当前栈中不存在该Activity的实例时，其行为同standard启动模式

#### SingleTask（栈内复用 taskAffinity属性对其有影响）

singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈

1.如果任务栈不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去

2.如果任务栈存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例，如果不存在对应activity的实例，则创建直接添加。

3.如果任务栈存在，则得到该任务栈，如果存在当前activity的实例，则会将当前activity之上的所有的activity出栈，并调用其**onNewIntent**方法
              
4.此外，我们可以将**两个不同App中的Activity设置为相同的taskAffinity**，这样虽然在不同的应用中，但是Activity会被分配到同一个Task中去。

#### SingleInstance（全局唯一模式）

该模式具备singleTask模式的所有特性外，与它的区别就是，这种模式下的Activity会单独占用一个Task栈，具有全局唯一性，即整个系统中就这么一个实例，由于栈内复用的特性，后续的请求均不会创建新的Activity实例，除非这个特殊的任务栈被销毁了。以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activity时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。
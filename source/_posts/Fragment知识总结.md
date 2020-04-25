---
title: Fragment知识总结
tags:
  - blog
categories:
  - Android
toc: false
date: 2020-04-25 16:18:43
---

### Fragment FragmentTransaction,FragmentManager，FragmentManagerImp的关系

**getSupportFragmentManager()** ： FragmentActivity中的方法，获取的是当前Activity创建的FragmentManagerImpl对象，该对象是所有该Activity中所有Fragment的管理类，也可以说是Fragment的入口。

**getChildFragmentManager（）**： Fragment中的方法，获取的是当前Fragment创建的FragmentManagerImpl对象，该对象用于管理所当前Fragment的所有子Fragment

**getFragmentManager()**：Fragment中的方法，获取的是add,remove,replace等方法传进去的FragmentManagerImpl对象，用于获取对应FragmentTransaction的信息

**beginTransaction()**：FragmentManager中的方法，返回FragmentTransaction对象，用于当前控制Fragment的添加，替换，删除等操作以及生命周期的控制。




### Fragment生命周期
::: hljs-center

![4625401a2c6ab45dc165d6b.webp](/images/2020/04/25/50105410-86c8-11ea-b594-476d70febf8d.webp)

:::
由于fragment的生命周期依赖于Activity，所以其生命周期与Activity及其相似，然其实现方式与Activity实现完全不同。

### Fragment生命周期的实现

在Fragment中并没有Activity对应的那么多状态，在Fragment中只有以下五个对应的状态：
![image.png](/images/2020/04/25/b4f8db40-86d7-11ea-b594-476d70febf8d.png)

**INITIALIZING** ： 初始状态，当前Fragment尚未创建

**CREATED** ： 已经创建状态，此时的生命周期路径为：

**ACTIVITY_CREATED**：创建完毕，但是尚未执行onStart

**STARTED**：执行了onStart尚未Resume

**RESUMED**：执行完毕，最终的显示状态

其中缺失了PAUSE,STOP ,DESTORY等状态，下面来分析。

Fragment的生命周期与Activity的生命周期一样，以onResume为分水岭，分为两段处理

#### onResume之前的逻辑
	

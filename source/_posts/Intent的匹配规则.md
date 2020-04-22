---
title: Intent的匹配规则
tags: []
categories:
  - Android
toc: false
date: 2020-04-19 09:41:09
---

### intent-filter的结构
![image.png](/images/2020/04/19/7d298210-81fe-11ea-92f8-ab273354db60.png)

### Intent被用于四大组件中出ContentProvider的其余三大组件（Activity，Service，BroadcastReceiver）

#### 知识点：
1.应用安装完成之后，系统会读取manifest.xml中的所有标签，将其一一分类保存

2.intent分为显式与隐式，显式Intent是直接指定组件的class（全路径），隐式则需要在manifest.xml文件的组件中添加intent-filter，使用时需要进行intent-filter的匹配

3.intent如果指定了组件的class，则为显式调用，即便manifest文件中配置了intent-filter,也不会再进行隐式的匹配

4.一个组件（Activity，Service，BroadcastReceiver）中可以有多个intent-filter标签

5.一个intent-filter标签中可以有多个category，多个action

### 匹配规则

6.如果intent中配置了category，则intent-filter中必须存在category且必须完全匹配才能验证通过

7.如果intent中没有设置category，则在匹配时系统会为intent默认添加一个android.intent.category.DEFAULT类型的category，**这也就是为什么要实现隐式调用的组件必须要添加一个android.intent.category.DEFAULT的原因**

8.intent-filter中可以有多个category，intent中的category只要能匹配intent-filter中的任意一个便可匹配成功

9.如果是隐式调用，则intent中必须要添加action，否则会匹配失败

10.intent-filter中可以有多个action，intent中的action只要能匹配intent-filter中任意一个action，便可以匹配成功

11.intent的data由MIME Type类型与URI组成

12.只有当intent-filter中的data标签不为空时才需要进行匹配，否则直接匹配成功。比如intent-filter中指定了mimeType则只需要匹配intent中mimeType类型

13.有一种特殊情况，如果intent-filter中指定了类型，但是intent中没有指定类型，但是却指定了URI，并且通过URI能推断出当前的类型，且推断出的类型能与intent-filter的类型匹配，此时依旧能匹配成功

### 结论

**14.Category，action，data三个共同影响intent-filter的匹配结果，只有三项完全匹配才算匹配成功，其中任意一个匹配失败，则当前intent-filter匹配失败**

15.extra是通过bundle的存储键值对，由于bundle实现了parcelable接口，所以能跨进程传输

16.Flags是能影响组件启动的各种标志位，具体的详见Activity的启动
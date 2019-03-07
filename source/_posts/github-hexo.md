---
title: 利用github + Hexo搭建自己个人博客的踩坑过程
tags:
  - blog
categories:
  - Hexo
  - Next
  - github
toc: false
date: 2019-03-07 11:02:47
---

其实一直以来都有弄一个自己私人博客的想法，但一直没有付诸行动。其实前段时间也有在简书与掘金写过博客，但是总觉得那些博客弄得花里胡哨，不是自己想要的。最终决定搭建自己的博客系统是在两天前，有一个自己的安心写东西的地方，也好记录自己成长的脚印。

下面就分享一下自己通过HEXO + github搭建自己个人博客的**踩坑**与折腾过程（后续效果优化会持续更新） 
搭建环境：windows 10
<!--more-->

1.首先，我们搭建的基本原则是将个人博客文件上传到github，然后进行访问，所以首先需要注册一个github账号，新建一个repository
![image.png](/images/2019/03/07/5fe5c8f0-408a-11e9-9621-77018395f2ce.png)

如果仓库的名字与github不一致，则在设置界面下面的东西需要修改，选择master branch，且访问路径需要带上全程，即"http://github.com//xxxx.github.io"

![image.png](/images/2019/03/07/c0f6b910-408a-11e9-9621-77018395f2ce.png)


2.开始安装所需环境（**git，nodejs**）

nodejs的下载路径：[Nodejs](https://nodejs.org/en/)

git的下载路径：[Git for Windows. 这里提供一个国内的下载站，方便网友下载](https://github.com/waylau/git-for-win)

当git安装完成之后需要配置git的全局用户名与邮箱（这里用户名就是你的github的用户名，邮箱为github注册邮箱）

打开git bash 输入的命令行为：
```
git config --global user.name "你的github名称"
git config --global user.email "你的github注册邮箱"
```

如下图所示：
![image.png](/images/2019/03/07/5e52b390-408f-11e9-9621-77018395f2ce.png)

3.安装hexo运行环境

命令行： **npm install -g hexo-cli** （如果找不到npm命令需要去配置环境变量）

安装时间可能有点长，稍微等待一下

4.hexo安装完成之后要开始初始化，切换到你博客想要放置的文件夹比方说 F：。然后执行以下指令：
hexo init xxxx（文件夹名），执行完成之后文件夹内容会如下所示：
```
INFO  Start blogging with Hexo!
```
![image.png](/images/2019/03/07/14a5b330-4091-11e9-9621-77018395f2ce.png)

因为你初始化hexo 之后source目录下自带一篇hello world文章, 所以直接执行下方命令
$ **hexo generate**
启动本地服务器
$ **hexo server**
在浏览器输入 http://localhost:4000/就可以看见网页和模板了
```
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
此时你可以通过http://localhost:4000/看到你博客的样子了，接下来我们要将它部署到github，然后通过github来访问

5.部署到github，部署到github先要来设置一个ssh key这个key的生成使用以下的指令：

**ssh-keygen -t rsa -C "Github的注册邮箱地址"**

一路Enter过来就可以了，最后会得到信息：
```
Your public key has been saved in /c/Users/user/.ssh/id_rsa.pub.
```
打开该文件，复制里面的全部内容，打开github的setting然后类似如下操作：

![image.png](/images/2019/03/07/aeb6a8c0-4092-11e9-9621-77018395f2ce.png)
title可以随便填写。

添加这个ssh key的意义在于如果是在别的电脑上想要down你项目的内容是会有重新验证的。

6.配置博客的信息，打开 **_config.yml** 文件，修改参数信息，里面有很多内容，可以根据自己的需要进行配置，以后有机会再来慢慢研究。
```
特别提示：特别提醒，在每个参数的：后都要加一个空格
```

我主要配置了以下几项：

```
title: 【临江仙（BrokenO)】
subtitle: Stay hungry, Stay foolish
description: Talking is cheap,show me your code 
keywords:
author: 【临江仙（BrokenO)】
language: zh-CN #zh-Hans（next主题用到的）
timezone: Asia/Shanghai
.
.
.
deploy:
  type: git
  repo: https://github.com/BrokenO/BrokenO.github.io.git(这个要配置成你自己的)
  branch: master
```

7.发表文章

命令行： hexo new "崔斯特测试文章"  （这个是新建一篇博客）

找到该文章，打开，使用Markdown语法，随便编辑几句保存

然后切换到博客的根目录执行以下命令：
```
F:\test\blog
$ hexo clean //先清空本地缓存
INFO  Deleted database.
INFO  Deleted public folder.

F:\test\blog 
$ hexo generate //生成静态文件
INFO  Start processing
INFO  Files loaded in 1.48 s
#省略
INFO  29 files generated in 4.27 s

F:\test\blog
$ hexo server //启动本地浏览服务器
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

此时博客已经通过http://localhost:4000/进行预览了

8.发布到网上

```
F:\test\blog
$ hexo deploy //发布到网上
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
#省略
```
#### 下面来说一下数据迁移的方式

具体的迁移方式详见：[创建Git分支将Hexo博客迁移到其它电脑](https://blog.csdn.net/White_Idiot/article/details/80685990)

#### 最后来说说git提交的三部曲
1. 本地发生改变之后，利用 **`git status`** 查看当前的改变情况
2. 利用 **``git add .``** 将当前的改变添加到缓存区
3. 指令 **``git commit -m "注释"``** 提交文件
4. **``git push origin xxxx(分支名)``** 更新到分支
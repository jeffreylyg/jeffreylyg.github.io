---
layout: post
title: "IntelliJ IDEA导入Android源码工程(AOSP)"
date: 2014-12-27 21:16:15 +0800
comments: true
categories: Android
---
这篇Blog主要介绍一下在IntelliJ IDEA中如何导入整个Android源码工程。

首先，确保整个Android源码被编译成功了（至于不知道如何下载Android源码和编译源码的同学请自行Google），在out目录中所对应的机型目录下成功生成了**system.img**这个文件。接下来就是如何导入的详细步骤了：

> 请注意：接下来的命令都是在你所下载的Android源码的根目录中执行。

<!--more-->

### 1. 执行如下命令生成idegen.jar文件

```
mmm developent/tools/idegen/ -B
```

### 2. 按如下命令执行idegen.sh脚本

```
development/tools/idegen/idegen.sh
```
这时如果是导入Android 5.0的源码的话会碰到如下图的问题：

{% img /images/2014-12-27/1.png %}

解决办法很简单，先将那个目录随便改个后缀名，等导入完成后改回来就好了。这条命令执行完成后就在Android源码的根目录中生成了android.iml, android.ipr和android.iws三个文件。

### 3. 用IntelliJ IDEA直接打开android.ipr这个文件

这时IntelliJ IDEA就会扫描整个目录并去建立索引了，这个过程是相当耗时间的，大概得一个小时左右，根据电脑的配置决定，所以为了能最大程度地缩短这个时间，在这之前我们有一些准备工作要做。

#### 增加IntelliJ IDEA所占的内存
如果是Linux，就在"IDEA_HOME/bin/idea.vmoptions64"（如果是mac系统就在"IntelliJ IDEA.app/Contents/Info.plist"）中改变""-Xms -Xmx"的值，如果是32位系统就修改vmoptions。我的电脑是16G的内存，所以我给Xmx开了1G。

### 4.一些后续步骤
当建立完索引后打开project structure就会在modules中看到如下图所示的很多jar文件的依赖：

{% img /images/2014-12-27/2.png %}

这时将除了Module Source 和 1.6 (No libraries)的jar文件都移除了，然后选Module SDK为一个空的java library（官方的说法），其实选对应版本的Android SDK也是没有任何问题的。

IntelliJ IDEA导入Android源码的步骤就这么多，接下来就可以在IntelliJ IDEA中自由自在地阅读源码或进行源码的开发了。





 

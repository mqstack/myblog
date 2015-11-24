title: Android构建与配置Gradle脚本综述
date: 2015-11-21 16:25:41
categories:
- Android
tags:
- Android
- Gradle
---
##基础

####Android构建过程

Gradle是以Groovy语言为基础的自动化构建工具。Android的构建系统是建立在Gradle基础上的。

什么是构建系统(Build System)？Android构建系统是一个工具集，可以用来测试、运行和打包你的app。我们可以从Android studio的菜单上来make project或者直接用命令行。简单的讲，构建的过程就是将源码编译，并将资源文件一起打包并签名的过程。

![构建过程图片](http://7xoljq.com1.z0.glb.clouddn.com/15-11-24/19933796.jpg)

如图所示，

##参考
1.https://developer.android.com/sdk/installing/studio-build.html
2.http://tools.android.com/tech-docs/new-build-system/user-guide
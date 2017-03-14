title: Android多媒体之一：编译ffmpeg
date: 2015-11-26 20:04:15
categories:
- Android
- FFmpeg
tags:
- Android
- FFmpeg
- Ndk
---
FFmpeg是功能强大的多媒体编解码库，广泛应用于各个平台的主流播放器、转码等软件。在Android框架对视频播放、编解码的支持没有那么强大时，使用ffmpeg也是不二的选择。本文介绍使用ndk编译ffmpeg的过程。

在看了很多人写的编译方法，尝试了很多方案后，发现只有下面这个方法能够一次成功，其他都会出各种各样的问题。
<!--more-->

## 准备
环境linux，

下载ffmpeg源码： http://www.ffmpeg.org/download.html

下载linux版ndk： http://developer.android.com/ndk/downloads/index.html

将源码和ndk都下载放到本地目录，路径随意。

## 写编译脚本
进入到ffmpeg目录，打开configure文件，找到以下几行：

	SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
	LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
	SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
	SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'

替换为

	SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
	LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
	SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
	SLIB_INSTALL_LINKS='$(SLIBNAME)'
这几行代码替换的原因是原来编译后的名称类似如下这种libavcodec.so.56，这样android编译时识别不出，所以换成类似libavcodec-56.so这样。

在ffmpeg目录下添加编译脚本build_android.sh，脚本中写入以下代码。脚本中主要是执行configure，然后执行make。

	#!/bin/bash
	NDK=$HOME/Desktop/adt/android-ndk-r9
	SYSROOT=$NDK/platforms/android-9/arch-arm/
	TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64
	function build_one
	{
	./configure \
	    --prefix=$PREFIX \
	    --enable-shared \
	    --disable-static \
	    --disable-doc \
	    --disable-ffmpeg \
	    --disable-ffplay \
	    --disable-ffprobe \
	    --disable-ffserver \
	    --disable-avdevice \
	    --disable-doc \
	    --disable-symver \
	    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
	    --target-os=linux \
	    --arch=arm \
	    --enable-cross-compile \
	    --sysroot=$SYSROOT \
	    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
	    --extra-ldflags="$ADDI_LDFLAGS" \
	    $ADDITIONAL_CONFIGURE_FLAG
	make clean
	make
	make install
	}
	CPU=arm
	PREFIX=$(pwd)/android/$CPU
	ADDI_CFLAGS="-marm"
	build_one

其中configure的参数加了如下一些：

prefix 指定了编译结果的目录；

enable 和disable 指定了需要编译的项；

cross-prefix指定了交叉编译的工具链中gcc文件；

target-os 不用说，是目标操作系统；

arch cpu类型；

sysroot androdlib目录。

更多conigure参数可参考./configure --help。


然后修改build_android.sh文件的权限：

	sudo chmod +x build_android.sh

并执行：

	./build_android.sh

接下来就等着吧，几分钟后便看到结果。

执行完毕后在android目录下就生成了lib和include目录，lib目录下面放的so文件，include目录下放的头文件。
将这些文件拷贝出来就可以准备后续开发了。

## 更多

[Android多媒体之二：jni调用ffmpeg命令](/2015/11/27/call-ffmpeg-with-jni/)
[Android多媒体之三：编译并使用x264库](/2015/12/03/build-x264-with-ndk/)

## 参考
http://www.roman10.net/how-to-build-ffmpeg-with-ndk-r9/

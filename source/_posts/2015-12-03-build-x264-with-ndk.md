title: Android多媒体之三：编译并使用x264库
date: 2015-12-03 17:58:34
categories:
- Android
tags:
- Android
- FFmpeg
- x264
---
x264是性能最好的H.264/AVC编码器，基于GNU GPL协议。FFmpeg可以使用x264作为编码库，能够提高编码性能。但注意ffmpeg是基于LGPL协议的，由于GPL协议的传染性，如果是使用ffmpeg的商业软件，需要确保没有使用x264等GPL协议的库。
<!--more-->

##准备
环境linux，

下载ffmpeg源码： http://www.ffmpeg.org/download.html

下载x264源码： git://git.videolan.org/x264.git

下载linux版ndk： http://developer.android.com/ndk/downloads/index.html

将源码和ndk都下载放到本地目录，路径随意。

##编译脚本

首先，跟ffmpeg一样，还是先进入到x264目录，修改configure文件。定位到
	
	else
		echo "SOSUFFIX=so" >> config.mak
		echo "SONAME=libx264.so.$API"

这几行，将"libx264.so.$API"替换为"libx264.so"，文件名中就不要版本了。

在目录下新建build_android_arm.sh脚本，写入以下代码：

	SYSROOT=$NDK/platforms/android-9/arch-arm/
	TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64
	function build_one
	{
	./configure \
	    --prefix=$PREFIX \
	    --enable-shared \
	    --disable-static \
	    --enable-pic \
	    --disable-asm \
		--disable-cli \
		--disable-pthread \
	    --host=arm-linux \
	    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
	    --sysroot=$SYSROOT \
	make clean
	make
	make install
	}
	PREFIX=$(pwd)/android/$CPU 
	build_one

具体跟编译ffmpeg类似，编译过ffmpeg的应该一看就懂了。

然后修改build_android_arm.sh文件的权限：

	sudo chmod +x build_android_arm.sh

并执行：

	./build_android.sh

一会，便在android/arm目录下看到编译结果了。

##编译包含x264的ffmpeg

之前一篇介绍了编译ffmpeg方法：

[Android多媒体之一：编译ffmpeg](/2015/11/26/android-multimedia-compile-ffmpeg/)

要加入x264，只需修改一点脚本。在之前脚本基础上在头文件和库的路径中加入x264的编译结果，然后configure的参数中支持x264就可以了。具体如下：

	SYSROOT=$NDK/platforms/android-9/arch-arm/
	TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64
	function build_one
	{
	./configure \
		--prefix=$PREFIX \
		--enable-shared \
		--disable-static \
		--enable-nonfree \
		--enable-gpl \
		--enable-asm \
		--disable-doc \
		--disable-ffmpeg \
		--disable-ffplay \
		--disable-ffprobe \
		--disable-ffserver \
		--disable-avdevice \
		--disable-symver \
		--enable-libx264 \
		--enable-encoder=libx264 \
		--enable-decoder=h264 \
		--enable-protocol=rtp \
		--enable-zlib \
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
	ADDI_CFLAGS="-I$(pwd)/x264/include/"
	ADDI_LDFLAGS="-L$(pwd)/x264/lib/"
	build_one

然后将编译好的include和lib目录拷贝至ffmpeg/x264目录下，执行脚本就可以了。

##更多

[Android多媒体之一：编译ffmpeg](/2015/11/26/android-multimedia-compile-ffmpeg/)
[Android多媒体之二：jni调用ffmpeg命令](/2015/11/27/call-ffmpeg-with-jni/)



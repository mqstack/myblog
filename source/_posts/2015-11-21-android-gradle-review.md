title: Android构建与配置Gradle脚本综述
date: 2015-11-21 16:25:41
categories:
- Android
tags:
- Android
- Gradle
---

## 基础

### Android构建过程

Gradle是以Groovy语言为基础的自动化构建工具。Android的构建系统是建立在Gradle基础上的。

什么是构建系统(Build System)？Android构建系统是一个工具集，可以用来测试、运行和打包你的app。我们可以从Android studio的菜单上来make project或者直接用命令行。简单的讲，构建的过程就是将源码编译，并将资源文件一起打包并签名的过程。
<!--more-->

![构建过程图片](http://7xoljq.com1.z0.glb.clouddn.com/15-11-24/19933796.jpg)

如图所示，构建的简要过程如下：



- Asset打包工具-aapt将所有的资源文件，例如AndroidManifest和其他xml文件编译，并且生成R.java提供给java编译器

- aidl工具将aidl接口转换为java接口

- java编译器将R.java，源码和aidl转换后的接口编译成 .class文件

- dex工具将编译后的 .class文件和第三方的一些库一起转换为Dalvik字节码

- apkbuilder工具将 .dex文件、编译好的资源文件及图片等不需要编译的资源文件打包成apk

- jarsigner将apk文件签名

- 最后zipalign工具优化apk，zipalign工具主要是用来保证未压缩的数据相对于文件初始位置从一个特定的位置开始读取。所有的未压缩数据例如图片和raw文件，都会对齐到4-byte边界。

### Gradle脚本基础

Android Studio新建一个工程后，会创建一个build.gradle文件，并且在每个module下面也会有一个build.gradle文件。

一个工程的默认配置文件如下，文件中指定了jcenter为依赖的仓库，并且指定了gradle的版本。

	buildscript {
	    repositories {
	        jcenter()
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:1.3.0'

		// NOTE: Do not place your application dependencies here; they belong
	    // in the individual module build.gradle files
		}
	}

	allprojects {
	    repositories {
	        jcenter()
	    }
	}

一个module的默认配置文件如下，
	apply plugin: 'com.android.application'

	android {
	    compileSdkVersion 23
		buildToolsVersion "23.0.2"

		defaultConfig {
		applicationId "org.mqstack.myapplication"
		minSdkVersion 15
		targetSdkVersion 23
		versionCode 1
		versionName "1.0"
		}
	    buildTypes {
	        release {
			minifyEnabled false
			proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			}
	    }
	}

	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
		compile 'com.android.support:appcompat-v7:23.1.0'
		compile 'com.android.support:design:23.1.0'
	}

其中：
> apply plugin 表明当前module是一个app，如果是library则写成 apply plugin: 'com.android.library'；

> android {...}	配置了android相关的构建选项：

> compileSdkVersion	编译的sdk版本

> buildToolsVersion	构建工具版本

> defaultConfig	配置了AndroidManifest中的一些参数，在构建时会覆盖AndroidManifest

> buildTypes	配置了如何构建app，默认有debug和release两种

> dependencies	表明了当前module依赖关系

一个module可以用三种方式依赖：

	dependencies {
	    // Module dependency
	    compile project(":lib")

	    // Remote binary dependency
	    compile 'com.android.support:appcompat-v7:19.0.1'

	    // Local binary dependency
	    compile fileTree(dir:'libs', include:['*.jar'])
	}


- 依赖统一工程下的另一个module

- 依赖远程库

- 依赖libs目录下的jar文件

好了，到这里应该能够看懂最简单的gradle脚本了，下一章介绍的知识能够帮助更灵活地配置。


## 进阶

下面以几个需求为例，介绍gradle灵活强大的自动构建能力。

### 构建不同的编译版本

我们知道，gradle脚本会默认提供两种构建方式：debug和release。

不过，如果需要出一个内部demo版本，需要有不一样的包名以便能够跟线上版本共存；需要不同的app名称以区分；也需要不同的版本号。这样的需求该如何实现呢？

可以通过配置productFlavors来实现。flavor的意思是口味、风味。意味着我们可以使用它构建不同特性的app，productFlavors中可以定义defaultConfig中的属性，构建的时候会覆盖这些属性。

比如可以新建一个demo flavor，可以替换applicationId、版本号等，使之与正常的app可以同时安装共存。

	productFlavors {
        demo {
            applicationId "com.buildsystemexample.app.demo"
            versionName "1.0-demo"
        }
        full {
            applicationId "com.buildsystemexample.app.full"
            versionName "1.0-full"
        }
    }

同时，src目录下，可以新建跟flavor同名的文件夹。文件夹跟main同级，里面可以未每个flavor指定不同的代码或者资源文件。构建的时候，指定的flavor里的代码和资源会覆盖main文件夹中的。

![](http://7xoljq.com1.z0.glb.clouddn.com/15-11-25/13888049.jpg)

例如可以在demo的Strings文件中修改app_name，这样demo flavor构建出的app，名字就变啦。

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <string name="app_name">app(demo)</string>
	</resources>

### 构建不同的渠道

使用不同的flavor可以构建不同的版本，那么如果需要使用友盟等工具统计渠道，该如何打包呢？

以友盟为例在Manifest中将value的值改为如下：

	<meta-data
	android:name="UMENG_CHANNEL"
	android:value="${UMENG_CHANNEL_VALUE}" />

我们在defaultConfig中添加placeholder，并给出默认值：

	defaultConfig {
	manifestPlaceholders = [UMENG_CHANNEL_VALUE: "umeng"]
	}

然后添加如下代码，可替换placeholder中的值为flavor的名称了，当然也可以替换为其他想要的名称。

	productFlavors.all { flavor ->
	    flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
	}

### 版本号中加时间

对于有dailybuild需求的团队，每天打的包如果版本名称一样，可能无法区分。可以在构建的版本号中加入日期：

	versionName = "1.0_" + buildTime()

	def buildTime() {
	def date = new Date()
	def formattedDate = date.format('yyyyMMdd')
	return formattedDate
	}

### 自定义apk文件名

gradle构建出的apk默认名称为“module的名字-debug”，我们可以在build脚本中指定构建出的apk名字。例如，以下代码将apk命名为：app名+版本名+flavor名。

	applicationVariants.all { variant ->
	    variant.outputs.each { output ->
			def fileName = "appname_v${variant.versionName}_release_${variant.flavorName}.apk"
			if (variant.buildType.isDebuggable()) {
		    	fileName = "appname_v${variant.versionName}_debug_${variant.flavorName}.apk"
			}
	        output.outputFile = new File(output.outputFile.parent, fileName)
	    }
	}

### 指定release keystore

为了安全，release keystore以及密码等最好只在编译机器上存放，脚本中不要直接写出路径和密码等字符。

这时候开发时会Android Studio会因为找不到RELEASE_STORE_FILE而报错，只需要在C:\Users\USER\.gradle中创建gradle.properties文件，并将四个变量设置为任意值就可以了。这样既不会影响工程中的gradle.properties版本，也能正常编译。

	release {
		storeFile file(RELEASE_STORE_FILE)
		storePassword RELEASE_STORE_PASSWORD
		keyAlias RELEASE_KEY_ALIAS
		keyPassword RELEASE_KEY_PASSWORD
	}

### 打包jar

有时候，我们新建的module可能是一个library，需要编译成jar包提供给项目使用，而gradle默认没有编译成jar文件而是aar文件。我们知道，jar是不包含资源文件例如图片，xml文件等。而相比较jar，aar能够将资源文件一同打包进去。

要打出jar包有以下两种方法：


1. 其实构建的时候已经产生了jar文件了，只不过藏得比较深而已。使用以下代码，将jar拷贝到libs文件夹下：

		task makeJar(type: Copy) {
		    delete 'build/libs/name.jar'
			from('build/intermediates/bundles/release/')
		    into('build/libs/')
		    include('classes.jar')
		    rename ('classes.jar', 'name.jar')
		}
		makeJar.dependsOn(build)


2. 也可以直接用下面的代码，将class文件打包成jar。

		android.libraryVariants.all { variant ->
			def name = variant.buildType.name
			def task = project.tasks.create "jar${name.capitalize()}", Jar
			task.dependsOn variant.javaCompile
			    task.from variant.javaCompile.destinationDir
			artifacts.add('archives', task);
		}

### 打包命令脚本

使用jenkins可以很方便持续集成，如果没有配置jenkins也可以单独写一个命令脚本，需要打release包的时候跑一下就行了。分享一个我们使用的命令脚本代码如下：

	if [ $# -lt 1 ]; then
	echo $0 ' '
	echo please input branch name.
	exit
	fi
	DIR=YOUR_DIR
	BRANCH=$1
	cd $DIR
	git fetch
	git checkout $BRANCH
	git pull origin $BRANCH
	#rm -f $DIR/YOUR_MODULE/build/outputs/apk/*.apk
	$DIR/gradlew clean assembleOneRelease assembleTwoRelease assembleThreeRelease


## 参考
1.https://developer.android.com/sdk/installing/studio-build.html
2.http://tools.android.com/tech-docs/new-build-system/user-guide

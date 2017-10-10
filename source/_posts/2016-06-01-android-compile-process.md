title: Android 编译打包流程及gradle插件开发
date: 2016-06-01 15:49:24
categories: 
- Android
tags:
- Android
- 编译
- gradle插件
---
我们编写APP代码后，点击运行按钮或者执行gradle命令就可以编译并打包应用了。但是源码究竟是如何变成最后的apk的呢？了解编译流程又能做些什么好玩的东西呢？
<!--more-->

我们编写APP代码后，点击运行按钮或者执行gradle命令就可以编译并打包应用了。但是源码究竟是如何变成最后的apk的呢？了解编译流程又能做些什么好玩的东西呢？

了解一些编译打包的流程及工作原理对工程师的开发技能和基础知识的提升都很有必要。所谓知其然，且知其所以然。当你遇到真正的需要制作工具、改进工具的需求时，当你遇到一些google搜索不到的问题需要自己分析时，不懂得原理只会使用工具寸步难行。

# 编译流程

## 典型的编译打包流程

一个典型的Android应用的编译流程如下：

![Compile process](https://developer.android.com/images/tools/studio/build-process_2x.png "Compile process")

可以看出，编译打包的过程大致有如下几个步骤：

	1.编译java源码和依赖包，转换为dex，编译资源文件；
	2.将dex和资源打包成apk；
	3.签名、zipalign优化。

这几个步骤结束后，就可以得到目标安装包了。

## gradle编译过程

我们知道，gradle的编译流程是由一个个task组成的。了解了编译过程中有哪些task，每个task都是做什么用的，那基本编译的流程就理解了。

如何知道task执行的流程呢？

新建一个Android工程，在build脚本中对project添加listener，可以打印出编译过程中执行的task。

listener 的代码如下：

```java
public class BuildTimeListener implements TaskExecutionListener, BuildListener {

    private Clock clock
    private times = []


    @Override
    public void buildStarted(Gradle gradle) {

    }

    @Override
    public void settingsEvaluated(Settings settings) {

    }

    @Override
    public void projectsLoaded(Gradle gradle) {

    }

    @Override
    public void projectsEvaluated(Gradle gradle) {

    }

    @Override
    public void buildFinished(BuildResult buildResult) {
        println "Task spend time:"
        for (time in times) {
            if (time[0] >= 50) {
                printf "%6sms %s\n", time
            }
        }
    }

    @Override
    public void beforeExecute(Task task) {
        clock = new Clock()
    }

    @Override
    public void afterExecute(Task task, TaskState taskState) {
        def ms = clock.timeInMs
        times.add([ms, task.path])
    }
}
```

添加listener的代码如下：
```java
project.gradle.addListener(new BuildTimeListener())
```

编译工程后会打印如下示例：

```
Task spend time:
   151ms :app:prepareComAndroidSupportAnimatedVectorDrawable2531Library
  1906ms :app:prepareComAndroidSupportAppcompatV72531Library
   127ms :app:prepareComAndroidSupportSupportCompat2531Library
    77ms :app:prepareComAndroidSupportSupportCoreUi2531Library
    50ms :app:prepareComAndroidSupportSupportFragment2531Library
   124ms :app:prepareComAndroidSupportSupportMediaCompat2531Library
    60ms :app:prepareComAndroidSupportSupportVectorDrawable2531Library
   122ms :app:compileDebugAidl
   332ms :app:generateDebugBuildConfig
   263ms :app:processDebugManifest
    69ms :app:dexPassDebugManifest
  7660ms :app:mergeDebugResources
  1288ms :app:processDebugResources
   112ms :app:incrementalDebugJavaCompilationSafeguard
 27512ms :app:compileDebugJavaWithJavac
   286ms :app:compileDebugNdk
    53ms :app:mergeDebugShaders
 12812ms :app:transformClassesWithDexForDebug
    59ms :app:transformNativeLibsWithStripDebugSymbolForDebug
   943ms :app:packageDebug
```

顾名思义，通过看这些task的名称能够猜到它的作用了：

1. compileDebugAidl 编译aidl接口，支持增量编译
2. generateDebugBuildConfig 生成buidconfig类
3. processDebugManifest 把所有依赖的aar中的manifest，合并到项目的AndroidManifest.xml中，并根据app中的buildType内容，替换manifest中的占位符，最后输出到app/build/intermediates/manifests/full/debug/AndroidManifest.xml。
4. mergeDebugResources 解压所有的aar包输出到app/build/intermediates/exploded-aar，并且把所有的资源文件合并到app/build/intermediates/res/merged/debug目录里
5. processDebugResources 1、调用aapt生成项目和所有aar依赖的R.java,输出到app/build/generated/source/r/debug目录
2、生成资源索引文件app/build/intermediates/res/resources-debug.ap_
3、把符号表输出到 app/build/intermediates/symbols/debug/R.txt
6. incrementalDebugJavaCompilationSafeguard IncrementalSafeguard 的作用是以生成的源码文件为输入，当任何源码文件变化时，我们清除AndroidJavaCompile的输出目录，并强制重新编译。
7. compileDebugJavaWithJavac 用来把java文件编译成class文件，输出的路径是app/build/intermediates/classes/debug
编译的输入目录有 1、项目源码目录，默认路径是app/src/main/java，可以通过sourceSets的dsl配置，允许有多个（打印project.android.sourceSets.main.java.srcDirs可以查看当前所有的源码路径,具体配置可以参考android-doc
2、app/build/generated/source/aidl
3、app/build/generated/source/buildConfig
4、app/build/generated/source/apt(继承javax.annotation.processing.AbstractProcessor做动态代码生成的一些库，输出在这个目录。
8. compileDebugNdk 编译ndk代码
9. mergeDebugShaders 编译shaders
10. transformClassesWithDexForDebug 这个任务的作用是把包含所有class文件的jar包转换为dex，class文件越多转换的越慢
输入的jar包路径是app/build/intermediates/transforms/jarMerging/debug/jars/1/1f/combined.jar
输出dex的目录是build/intermediates/transforms/dex/debug/folders/1000/1f/main
11. transformNativeLibsWithStripDebugSymbolForDebug 
12. packageDebug 打包apk

需要注意的是，每个版本的buildtool中的编译流程是大致相同，但是task任务都是有变化的。比如最近的2.x版本增量编译做的越来越好，编译时间大大减少了，通过打印task会发现有很多增量编译的任务。

# 应用

了解了编译的流程，可以做些什么呢？很多常见的工具优化都是通过在编译期执行自定义代码实现的，比如打渠道包、增量编译、热更新等。

我们来尝试下，如何自定义一个task，并在指定的时机去执行。

## 首先，怎么才能在想要的时机去执行特定的任务呢？

在这里用到task的两个方法：
	
	mustRunAfter：表示当前task在某个task后执行
	dependsOn：表示当前task依赖与某个task
用这两个方法可以将我们的task插入到想要的两个task中间，例如：
 ```
 manifestTask.mustRunAfter variantOutput.processManifest
 variantOutput.processResources.dependsOn manifestTask
```

## 其次，我们的插件怎么应用到工程中呢？

跟其他插件一样，用
	
	apply plugin:

就可以了。

当依赖了插件以后，编译时会执行Plugin中的apply方法。只要将执行任务的代码放到apply方法中，就可以执行了。

```java
class XXXPlugin implements Plugin<Project> {
	void apply(Project project) {
		//Add your code here
	}
}
```

## 然后，插件还支持扩展

通过扩展，可以很方便的在build脚本中向插件传递参数。例如，向application插件中传递sdk版本参数：

```java
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    ...
}
```
要实现扩展也很简单，自定义一个类：

```java
public class XXXExtension {

    boolean xxxEnable = true

    String inputMessage = "Default Message"

    int version = 1
}
```

然后在plugin中执行就可以了：

```java
project.extensions.create('xxxplugin', XXXExtension)
```
读取扩展参数的代码如下：

```java
def config = project.xxxplugin
```

## 最后，我们实现一个简单的例子

编写一个常见的打渠道包的插件。其原理就是应用上述的方法，通过参数扩展定义渠道包名，并在编译期间通过修改manifest内容实现打渠道包。

具体代码参考

https://github.com/mqstack/DexPass


# 引用

http://www.jianshu.com/p/53923d8f241c

https://developer.android.com/studio/build/index.html#build-config



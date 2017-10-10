# Android

## 后台唤醒
广播唤醒：
静态广播，杀掉的app无法接收；可以通过指定context和classname方式;
接收广播方的app, android:exported="true"
```java
Intent intent = new Intent();
Context c = null;
try {
    c = createPackageContext("com.example.broadcasttest", Context.CONTEXT_INCLUDE_CODE | Context.CONTEXT_IGNORE_SECURITY);
} catch (PackageManager.NameNotFoundException e) {
    e.printStackTrace();
}
intent.setClassName(c, "com.example.broadcasttest.TestBroadcastReceiver");
intent.setAction("my.broadcast.test");
intent.setFlags(Intent.FLAG_INCLUDE_STOPPED_PACKAGES);
sendBroadcast(intent);
```
service唤醒：
serviceIntent.setPackage("com.yiba.test.yibaapp");
serviceIntent.setAction("com.yiba");
startService(serviceIntent);

## recyclerview 
RecyclerView 就是将 onMeasure()、onLayout() 交给了 LayoutManager 去处理，因此如果给 RecyclerView 设置不同的 LayoutManager 就可以达到不同的显示效果

ItemDecoration: RecyclerView 执行到 onDraw() 方法的时候，就会调用到他的 onDraw()，getItemOffsets()的方法，从字面就可以理解，他是用来偏移每个 item 视图的

ItemAnimator
们手动调用notifyxxxx()的时候。通常这个时候我们会要传一个下标，那么从这个标记开始一直到结束，所有 item 视图都会被执行一次这个动画

Adapter
与 ListView 不同的是，ListView 的适配器是直接返回一个 View，将这个 View 加入到 ListView 内部。而 RecyclerView 是返回一个 ViewHolder 并且不是直接将这个 holder 加入到视图内部，而是加入到一个缓存区域，在视图需要的时候去缓存区域找到 holder 再间接的找到 holder 包裹的 View。

ViewHolder
 ViewHolder 的内部是一个 View，并且 ViewHolder 必须继承自RecyclerView.ViewHolder类。 这主要是因为 RecyclerView 内部的缓存结构并不是像 ListView 那样去缓存一个 View，而是直接缓存一个 ViewHolder ，在 ViewHolder 的内部又持有了一个 View

ViewHolder三级缓存
第一级缓存：
就是上面的一系列 mCachedViews。如果仍依赖于 RecyclerView （比如已经滑动出可视范围，但还没有被移除掉），但已经被标记移除的 ItemView 集合会被添加到 mAttachedScrap 中。然后如果 mAttachedScrap 中不再依赖时会被加入到 mCachedViews 中。 mChangedScrap 则是存储 notifXXX 方法时需要改变的 ViewHolder 。

第二级缓存：
ViewCacheExtension 是一个抽象静态类，用于充当附加的缓存池，当 RecyclerView 从第一级缓存找不到需要的 View 时，将会从 ViewCacheExtension 中找。不过这个缓存是由开发者维护的，如果没有设置它，则不会启用。通常我们也不会去设置他，系统已经预先提供了两级缓存了，除非有特殊需求，比如要在调用系统的缓存池之前，返回一个特定的视图，才会用到他。

第三级缓存：
最强大的缓存器。之前讲了，与 ListView 直接缓存 ItemView 不同，从上面代码里我们也能看到，RecyclerView 缓存的是 ViewHolder。而 ViewHolder 里面包含了一个 View 这也就是为什么在写 Adapter 的时候 必须继承一个固定的 ViewHolder 的原因
缓存池，实现上，是通过一个默认为 5 大小的 ArrayList 实现的。这一点，同 ListView 的 RecyclerBin 这个类一样
每一个 ArrayList 又都是放在一个SparseArray里

## 嵌套滑动
NestedScrollingParent
onStartNestedScroll该方法，一定要按照自己的需求返回true，该方法决定了当前控件是否能接收到其内部View(非并非是直接子View)滑动时的参数；假设你只涉及到纵向滑动，这里可以根据nestedScrollAxes这个参数，进行纵向判断。
onNestedPreScroll该方法的会传入内部View移动的dx,dy，如果你需要消耗一定的dx,dy，就通过最后一个参数consumed进行指定，例如我要消耗一半的dy，就可以写consumed[1]=dy/2
onNestedFling你可以捕获对内部View的fling事件，如果return true则表示拦截掉内部View的事件。

## fragment状态与重建
一旦 Fragment 从回退栈（BackStack）中返回时，View 将会被销毁和重建。Fragment 没有被销毁，但 Fragment 的 View 被销毁
自定义view，实现onSaveInstanceState 和 onRestoreInstanceState 方法。


## davik art
Dalvik与JVM

Dalvik是Android平台的虚拟机,在应用每次运行的时候会 将 dex文件 编译为 机器码。
Dalvik使用.dex格式，而Java虚拟机使用的是class格式
一个dex文件可以包含若干个类
Dalvik指令是基于寄存器的，Java虚拟机使用的指令集是基于堆栈的
自Android 2.2开始，Dalvik支持JIT技术

ART

是一种在Android操作系统上的运行环境，ART能够在第一次安装的时候，把应用程序的字节码转换为机器码。采用了预编译（AOT,Ahead-Of-Time）技术。
ART运行的仍然是一个包含dex字节码的APK文件；
AOT在在应用安装的时候将应用的dex字节码翻译成本地机器码；
ART的优点

ART 性能高于采用JIT的Dalvik
应用启动更快、运行更快、体验更流畅、触感反馈更及时
更长的电池续航能力
支持更低的硬件
ART的缺点

字节码变为机器码之后，占用的存储空间更大
应用的安装时间会变长。


## 进程通信
linux使用的进程间通信方式
管道（pipe）,流管道(s_pipe)和有名管道（FIFO）
信号（signal）
消息队列
共享内存
信号量
套接字（socket)

Android：
通过Intent在Activity、Service或BroadcastReceiver间进行进程间通信，可通过Intent传递数据
AIDL方式
Messenger方式
利用ContentProvider
Socket方式
基于文件共享的方式

## 协程
多个线程相对独立，有自己的上下文，切换受系统控制；而协程也相对独立，有自己的上下文，但是其切换由自己控制，由当前协程切换到其他协程由当前协程来控制

由一个线程执行，produce和consumer协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

## launchmod
standard，创建一个新的Activity。
singleTop，栈顶不是该类型的Activity，创建一个新的Activity。否则，onNewIntent。
singleTask，回退栈中没有该类型的Activity，创建Activity，否则，onNewIntent+ClearTop。
singleInstance，回退栈中，只有这一个Activity，没有其他Activity。

## 绘制
measure 过程传递尺寸的两个类
ViewGroup.LayoutParams （View 自身的布局参数）
MeasureSpecs 类（父视图对子视图的测量要求）

在 layout 过程中，子视图会调用getMeasuredWidth()和getMeasuredHeight()方法获取到 measure 过程得到的 mMeasuredWidth 和 mMeasuredHeight，作为自己的 width 和 height。然后调用每一个子视图的layout(l, t, r, b)函数，来确定每个子视图在父视图中的位置。

draw：
1. Draw the background if need
2. If necessary, save the canvas' layers to prepare for fading
3. Draw view's content
4. Draw children (dispatchDraw)
5. If necessary, draw the fading edges and restore layers
6. Draw decorations (scrollbars for instance)

# ANR
应用在5秒内未响应用户的输入事件
BroadcastReceiver未在10秒内完成相关的处理
Service在特定的时间内无法处理完成 20秒

## 插件化
1. 代码修复
底层替换方案：已经加载的类中直接替换原有方法；
缺点，方法增加或减少，无法正常索引到正确的方法
不稳定，直接修改虚拟机方法实体的字段

类加载方案：
插桩，先加载补丁中的类
tinker，合成全量dex
sophix：重新编排包中dex顺序

2. 资源修复
InstantRun 两步修复：
构造新的AssetManager，通过反射调用addAsetPath，把新资源包加入到新AssetManager
找到所有引用AssetManager的地方，用反射替换为新的

sophix：
构造package id 为0x66的资源包，加入到assetpath中

3. so修复
把补丁so库插入到nativeLibraryDirectories前面，加载so库的时候是补丁



## Service
与activity通信，Binder
public void onServiceConnected(ComponentName name, IBinder service)
IBinder强转为对象

IntentService
onHandleIntent 执行任务

## Broadcast
普通广播:
是完全异步的，可以在同一时刻（逻辑上）被所有接收者接收到，所有满足条件的 BroadcastReceiver 都会随机地执行其 onReceive() 方法
在有序广播中:
同级别接收是先后是随机的,
我们可以在前一个广播接收者将处理好的数据传送给后面的广播接收者
高级别的广播收到该广播后，可以决定把该广播是否截断掉
粘性广播:
sendStickyBroadcast
发出的广播会一直滞留（等待），以便有人注册这则广播消息后能尽快的收到这条广播
粘性有序广播


## binder
进程：操作系统为一个二进制可执行文件创建了一个载有该文件自己的栈，堆、数据映射以及共享库的内存片段，还为其分配特殊的内部管理结构。
线程 ：线程是没有自己内存地址空间的过程，它与父进程共享内存地址空间。

IPC(Inter-Process Communication)
关于IPC我主要说明如下两点：
IPC是一种用于多进程间数据信号交换的框架；
IPC方式包括信息传递，数据同步，共享内存和远程过程调用等；

概念：
从代码角度来看，Binder 是一个类，实现了 IBinder接口；
从来源看，Binder 来自于 OpenBinder,是 Android IPC 机制中的一种，Binder 还可以理解成一个虚拟物理设备，设备驱动是 dev/binder；
从 Framework 层看，Binder 是 Service Manager 连接各种 Manager(ActivityManager,PackageManager...) 和相应 Service(ActivityManagerService,PackageManagerService...) 的桥梁；
从客户端看，Binder 是客户端服务器通讯的媒介

AIDL：
客户端和服务端放同一个aidl接口
服务端Service实现.Stub
客户端通过Inteng绑定service
通过ServiceConnection中IBinder iBinder参数调用服务端

## LruCache
LinkedHashMap存储 双向链表,记录的插入顺序，构造参数允许按访问次序排序
put和get都会加到尾部
trimsize，从头部开始删除

重写sizeOf方法



## android设计模式
对遇到的问题解决方案的抽象
1. 工厂：类的管理和实例化创建
举例：AsyncTask中ThreadFactory，需要知道线程总数量 bitmapfactory
2. 适配器模式，ListAdapter数据和视图
3. 责任链模式：View触摸事件传递
可以降低系统的耦合度（请求者与处理者代码分离），简化对象的相互连接，同时增强给对象指派职责的灵活性，增加新的请求处理类也很方便
4. 观察者模式 BaseAdapter-> DataSetObservable DataSetObserver
5. 建造者模式:
 客户端不必知道产品内部组成的细节
 具体的建造者类之间是相互独立的，对系统的扩展非常有利。
6. 备忘录模式 
onSaveInstanceState和onRestoreInstanceState就是通过Bundle（相当于备忘录对象）这种序列化的数据结构来存储Activity的状态
7. 策略模式: 通过组合将行为和策略分开，多态特性，设置不同的策略去调用Animation  setInterpolator
8. 代理模式: 跨进程通信Binder
9. 状态模式：
使用状态模式前，客户端外界需要介入改变状态，而状态改变的实现是琐碎或复杂的。
使用状态模式后，客户端外界可以直接使用事件Event实现，根本不必关心该事件导致如何状态变化，这些是由状态机等内部实现。
这是一种Event-condition-State，状态模式封装了condition-State部分。


# 性能
## 布局性能：
1. 降低层级
2. include和merge 减少层级
3. viewstub 延时加载，调用visibility或inflate加载
4. 后台线程--除了生命周期回调，UI渲染
5. viewholder--recyclerview中已经改进
6. 过度渲染

clipRect来解决自定义View的过度绘制
onDraw里分配内存易造成内存抖动

## 渲染
使用Androidstudio LayoutInspector来查找Activity中的布局是否过于复杂
也可以使用手机设置里面的开发者选项，打开Show GPU Overdraw等选项进行观察
你还可以使用TraceView来观察CPU的执行情况
Profile GPU Rendering 观察渲染速度

VSYNC信号：当屏幕从缓冲区扫描完一帧到屏幕上之后，开始扫描下一帧之前，发出的一个同步信号，该信号用来切换前缓冲区和后缓冲区

双缓冲：一个buffer显示同时另一个buffer准备数据，问题当gpu绘制延时时，SurfaceFlinger又占有一个buffer用于下一帧显示，没有buffer供cpu准备数据；

三缓冲：cpu gpu sufacefinger各占一个buffer
![](https://dn-mhke0kuv.qbox.me/556fb634ca4cf7713bd8.png)
![](https://dn-mhke0kuv.qbox.me/47a962b7b7e477e031f2.png)

## 内存
Young generation
old generation
permanent generation

gc时所有线程暂停

内存抖动：频繁创建和释放
Young Generation的每次GC操作时间是最短的，Old Generation其次，Permanent Generation最长。执行时间的长短也和当前Generation中的对象数量有关，遍历查找20000个对象比起遍历50个对象自然是要慢很多的。

Memory Monitor：查看整个app所占用的内存，以及发生GC的时刻，短时间内发生大量的GC操作是一个危险的信号。
Allocation Tracker：使用此工具来追踪内存的分配，前面有提到过。
Heap Tool：查看当前内存快照，便于对比分析哪些对象有可能是泄漏了的，请参考前面的Case。

## oom
1. 使用轻量数据结构
2. 避免enum
3. bitmap 内存占用
4. 对象重复利用
5. 避免ondraw创建对象
6. stringbuilder
7. 资源文件合适文件夹
8. 评估可能OOM的代码，加入try catch
9. 谨慎使用static
10. 单例对象中不合理持有
11. service使用后及时停止 建议使用intent service
12. 谨慎使用第三方libraries


## 内存泄露
1. activity的泄露 handler延迟任务，context传到其他实例中
2. 使用application context
3. 临时bitmap的回收
4. 监听器的注销
5. 缓存容器中的对象泄露
6. webview泄露， 
    ondestroy web_view_.removeAllViews();
    ((ViewGroup) web_view_.getParent()).removeView(web_view_);
    web_view_.setTag(null);
    web_view_.clearHistory();
    web_view_.destroy();
    web_view_ = null;
7. cursor对象及时关闭


## 运算性能
Traceview
HashMap ArrayMap
Autoboxing
The price of ENUM

## 性能优化
当程序被启动，系统会帮忙创建进程以及相应的主线程，而这个主线程其实就是一个HandlerThread。这个主线程会需要处理系统事件，输入事件，系统回调的任务，UI绘制等等任务，为了避免主线程任务过重，我们就会需要不断的开启新的工作线程来处理那些子任务

HandlerThread比较合适处理那些在工作线程执行，需要花费时间偏长的任务
asynctask 轻量级、短时间


## leakcanary 分析
对所有activity生命周期监听
application.registerActivityLifecycleCallbacks
WeakReference ReferenceQueue,发生 GC 后，WeakReference 所持有的对象如果被回收就会进入该队列
检测ReferenceQueue 是否有该activity判断是否内存泄露

用haha分析Android heap dump文件

## 安全

1. 本地数据加密，keygenerator AndroidKeyStore
2. https
3. key放到native中，验证keystore
    1. 签名证书文件校验码
```java
public static String getSign(Context ctx) {
    try {
        PackageInfo packageInfo = ctx.getPackageManager().getPackageInfo(ctx.getPackageName(),
                PackageManager.GET_SIGNATURES);
        Signature[] signs = packageInfo.signatures;
        Signature sign = signs[0];
        MessageDigest md1 = MessageDigest.getInstance("MD5");
        md1.update(sign.toByteArray());
        byte[] digest = md1.digest();
        String res = toHexString(digest);
        MessageDigest md2 = MessageDigest.getInstance("SHA1");
        md2.update(sign.toByteArray());
        byte[] digest2 = md2.digest();
        String res2 = toHexString(digest2);
        return res2;
    } catch (Exception e) {
        e.printStackTrace();
        return "";
    }
}
```
    2. 完整性校验
```java
public static void apkVerifyWithSHA(Context context, String baseSHA) {  
       String apkPath = context.getPackageCodePath(); // 获取Apk包存储路径  
       try {  
           MessageDigest dexDigest = MessageDigest.getInstance("SHA-1");  
            byte[] bytes = new byte[1024];  
            int byteCount;  
            FileInputStream fis = new FileInputStream(new File(apkPath)); // 读取apk文件  
            while ((byteCount = fis.read(bytes)) != -1) {  
                dexDigest.update(bytes, 0, byteCount);  
            }  
            BigInteger bigInteger = new BigInteger(1, dexDigest.digest()); // 计算apk文件的哈希值  
            String sha = bigInteger.toString(16);  
            fis.close();  
            if (!sha.equals(baseSHA)) { // 将得到的哈希值与原始的哈希值进行比较校验  
                Process.killProcess(Process.myPid()); // 验证失败则退出程序  
            }  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
        } catch (FileNotFoundException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
    }
```


# 算法
1. 冒泡排序：相邻两个元素，交换
2. 快排：
```java
public static int getMiddle(int[] numbers, int low,int high)
{
    int temp = numbers[low]; //数组的第一个作为中轴
    while(low < high)
    {
        while(low < high && numbers[high] > temp)
        {
            high--;
        }
        numbers[low] = numbers[high];//比中轴小的记录移到低端
        while(low < high && numbers[low] < temp)
        {
            low++;
        }
        numbers[high] = numbers[low] ; //比中轴大的记录移到高端
    }
    numbers[low] = temp ; //中轴记录到尾
    return low ; // 返回中轴的位置
}

public static void quickSort(int[] numbers,int low,int high)
{
    if(low < high)
    {
    　　int middle = getMiddle(numbers,low,high); //将numbers数组进行一分为二
    　　quickSort(numbers, low, middle-1);   //对低字段表进行递归排序
    　　quickSort(numbers, middle+1, high); //对高字段表进行递归排序
    }
}
```

3. 选择：选出最小的与第一个交换
4. 插入：将待排序的元素插入到正确位置
5. 希尔
6. 归并
7. 堆排序

# java

WeakReference ，一旦失去最后一个强引用，就会被 GC 回收，而软引用虽然不能阻止被回收，但是可以延迟到 JVM 内存不足的时候。

栈常用于保存方法帧和局部变量，而对象总是在堆上分配

编译期常量 static final，编译时会被替换成常量

List 是一个顺序集合，迭代按插入顺序；Set没有重复元素，是无序集合，迭代无序。

PriorityQueue 保证最高或者最低优先级的的元素总是在队列头部,当遍历一个 PriorityQueue 时，没有任何顺序保证。

StringBuilder 提供与StringBuffer兼容的api，但不同步。

## 虚拟机实现
内存管理，类加载，双亲委派

加载-》验证-》准备-》解析-》初始化-》使用-》卸载
### 类加载
http://blog.csdn.net/ns_code/article/details/17881581
启动类加载器-》扩展类加载器-》应用类加载器-》自定义类加载器
双亲委派模型，即我们把每一层上面的类加载器叫做当前层类加载器的父加载器，当然，它们之间的父子关系并不是通过继承关系来实现的，而是使用组合关系来复用父加载器中的代码。
双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

使用双亲委派模型来组织类加载器之间的关系，有一个很明显的好处，就是Java类随着它的类加载器（说白了，就是它所在的目录）一起具备了一种带有优先级的层次关系，这对于保证Java程序的稳定运作很重要。例如，类java.lang.Object类存放在JDK\jre\lib下的rt.jar之中，因此无论是哪个类加载器要加载此类，最终都会委派给启动类加载器进行加载，这边保证了Object类在程序中的各种类加载器中都是同一个类。

### 验证
文件格式的验证、元数据的验证、字节码验证和符号引用验证。

### 准备
 准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。
1、这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。

2、这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。

### 解析
解析阶段是虚拟机将常量池中的符号引用转化为直接引用的过程。
1、类或接口的解析：判断所要转化成的直接引用是对数组类型，还是普通的对象类型的引用，从而进行不同的解析。
2、字段解析：对字段进行解析时，会先在本类中查找是否包含有简单名称和字段描述符都与目标相匹配的字段，如果有，则查找结束；如果没有，则会按照继承关系从上往下递归搜索该类所实现的各个接口和它们的父接口，还没有，则按照继承关系从上往下递归搜索其父类，直至查找结束，查找流程如下图所示：
本类-》接口-》父接口-》。。。-》父类-》祖父类-》。。。

```java
class Super{  
    public static int m = 11;  
    static{  
        System.out.println("执行了super类静态语句块");  
    }  
}  
  
  
class Father extends Super{  
    public static int m = 33;  
    static{  
        System.out.println("执行了父类静态语句块");  
    }  
}  
  
class Child extends Father{  
    static{  
        System.out.println("执行了子类静态语句块");  
    }  
}  
  
public class StaticTest{  
    public static void main(String[] args){  
        System.out.println(Child.m);  
    }  
}  
```
执行结果如下：
执行了super类静态语句块
执行了父类静态语句块
33
如果注释掉Father类中对m定义的那一行，则输出结果如下：
执行了super类静态语句块
11

### 初始化
初始化阶段，则是根据程序员通过程序指定的主观计划去初始化类变量和其他资源，或者可以从另一个角度来表达：初始化阶段是执行类构造器<clinit>()方法的过程



## 内部类
内部类是面向对象的闭包，因为它不仅包含创建内部类的作用域的信息，还自动拥有一个指向此外围类对象的引用，在此作用域内，内部类有权操作所有的成员，包括private成员

每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

内部类可以大大弥补Java在多重继承上的不足。
另外匿名内部类可以方便的实现闭包。
静态内部类可以带来更好的代码聚合，提高代码可维护性。

## 静态方法
方法被声明为是static的，而静态方法是不能被覆写的；它们只能被隐藏。

## vector arraylist
ArrayList在内存不够时默认是扩展50% + 1个，Vector是默认扩展1倍。
Vector提供indexOf(obj, start)接口，ArrayList没有。
Vector属于线程安全级别的，但是大多数情况下不使用Vector，因为线程安全需要更大的系统开销。

SynchronizedList  Iterator需要自己同步
## TreeMap和TreeSet
红黑树实现，TreeSet基于TreeMap实现
相同点：

TreeMap和TreeSet都是有序的集合，也就是说他们存储的值都是拍好序的。
TreeMap和TreeSet都是非同步集合，因此他们不能在多线程之间共享，不过可以使用方法Collections.synchroinzedMap()来实现同步
运行速度都要比Hash集合慢，他们内部对元素的操作时间复杂度为O(logN)，而HashMap/HashSet则为O(1)。
不同点：
最主要的区别就是TreeSet和TreeMap非别实现Set和Map接口
TreeSet只存储一个对象，而TreeMap存储两个对象Key和Value（仅仅key对象有序）
TreeSet中不能有重复对象，而TreeMap中可以存在

## HashSet和HashMap
HashSet 的内部采用 HashMap来实现
所有 key 的都有一个默认 value
不允许重复key，只有一个null key

## HashMap 和 HashTable
Map 接口和 Dictionary 接口
前者线程不同步，在单线程条件下操作性能较好，后者线程同步，在多线程条件下可以正确操作，不会发生多线程下的操作问题。

使用Collections synchronizedMap，对HashMap做封装，加了synchronized

HashTable并发性不如ConcurrentHashMap，因为分段锁

## ArrayMap HashMap

## 泛型
Producer Extends Consumer Super

## Memory Mapped Files 
允许直接从内存中访问文件,速度比stream io快
实现方式是将整个或者部分文件映射到内存中，
操作系统负责加载页请求、写文件，应用只需要直接跟内存打交道，即使应用crash了，系统依然继续完成读写操作

```java
RandomAccessFile memoryMappedFile = new RandomAccessFile("largeFile.txt", "rw");
//Mapping a file into memory
MappedByteBuffer out = memoryMappedFile.getChannel().map(FileChannel.MapMode.READ_WRITE, 0, count);
//Writing into Memory Mapped File
for (int i = 0; i < count; i++) {
    out.put((byte) 'A');
}
System.out.println("Writing to Memory Mapped File is completed");
//reading from memory file in Java
for (int i = 0; i < 10 ; i++) {
    System.out.print((char) out.get(i));
}
System.out.println("Reading from Memory Mapped File is completed");
```

## 多线程
最小化同步的范围
更偏向于使用 volatile 而不是 synchronized
使用更高层次的并发工具
优先使用并发集合

volatile作用是使变量在多个线程间可见，作用是强制从公共堆栈中取得变量的值，
线程中的变量存在于公共堆栈及线程的私有堆栈中


### synchronized
它无法中断一个正在等候获得锁的线程；
也无法通过投票得到锁，如果不想等下去，也就没法得到锁；
同步还要求锁的释放只能在与获得锁所在的堆栈帧相同的堆栈帧中进行，多数情况下，这没问题（而且与异常处理交互得很好），但是，确实存在一些非块结构的锁定更合适的情况。

### ReentrantLock
可重入性：若一个程序或子程序可以“在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant或re-entrant）的。

从设计上讲，当一个线程请求一个由其他线程持有的对象锁时，该线程会阻塞。当线程请求自己持有的对象锁时，如果该线程是重入锁，请求就会成功，否则阻塞。


重入性在我看来实际上表明了锁的分配机制：基于线程的分配，而不是基于方法调用的分配。

添加了类似锁投票、定时锁等候和可中断锁等候的一些特性。此外，它还提供了在激烈争用情况下更佳的性能。
所以为了保证锁最终被释放(发生异常情况)，要把互斥区放在try内，释放锁放在finally内！！

a)  lock(), 如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁

b) tryLock(), 如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；

c)tryLock(long timeout,TimeUnit unit)，   如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时，返回false；

d) lockInterruptibly:如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到或者锁定，或者当前线程被别的线程中断

公平锁 排队
非公平锁 无法保证顺序

synchronized就是非公平锁
可以在创建ReentrantLock对象时，通过布尔参数来决定使用 非公平锁 还是公平锁


### ReadWriteLock
解决读和读不互斥的问题
允许对共享数据进行更高级别的并发访问
ReentrantReadWriteLock实现了这个接口

### Condition

final Lock lock = new ReentrantLock();          //锁对象  
final Condition notFull  = lock.newCondition(); //写线程锁  
final Condition notEmpty = lock.newCondition(); //读线程锁 
wait -> await
notify -> signal

## 内存模型

内存区域：
1. 方法区 存储已经被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。方法区域又被称为“永久代”
2. 堆区
3. 本地方法栈 为使用到的本地操作系统（Native）方法服务
4. 虚拟机栈 每个方法被执行的时候都会同时创建一个栈帧，栈它是用于支持续虚拟机进行方法调用和方法执行的数据结构，栈帧用于存储局部变量表、操作数栈、动态链接、方法返回地址和一些额外的附加信息

5. 程序计数器 当前线程所执行的字节码的行号指示器，字节码解释器工作时通过改变该计数器的值来选择下一条需要执行的字节码指令

其中，方法区和堆是所有线程共享的。

1. 分为两块heap 和 thread stack
2. thread stack包含调用栈信息、本地变量(可以是基本类型，也可以是对象，对象保存引用)
![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-2.png)
3. 硬件模型：寄存器、缓存、内存；与jvm架构不同，heap和stack都可能在缓存和寄存器中，两个问题
线程更新共享变量的可见性 volatile
共享变量读、写、检查有竞争条件 sychronized wait notify

重排序分三种类型：

编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

底层实现，内存屏障：
一旦内存数据被推送到缓存，就会有消息协议来确保所有的缓存会对所有的共享数据同步并保持一致。这个使内存数据对CPU核可见的技术被称为内存屏障或内存栅栏。
内存屏障提供了两个功能。首先，它们通过确保从另一个CPU来看屏障的两边的所有指令都是正确的程序顺序，而保持程序顺序的外部可见性；其次它们可以实现内存数据可见性，确保内存数据会同步到CPU缓存子系统。
Store Barrier  Load Barrier

## GC
引用计数算法：
引用和去引用伴随加法和减法，影响性能
致命的缺陷：对于循环引用的对象无法进行回收

根搜索算法：
说到GC roots（GC根），在JAVA语言中，可以当做GC roots的对象有以下几种：
1、栈（栈帧中的本地变量表）中引用的对象。
2、方法区中的静态成员。
3、方法区中的常量引用的对象（全局变量）
4、本地方法栈中JNI（一般说的Native方法）引用的对象。

垃圾搜集的算法主要有三种，分别是标记-清除算法、复制算法、标记-整理算法：
标记-清除算法的缺点：
（1）首先，它的缺点就是效率比较低（递归与全堆对象遍历），导致stop the world的时间比较长，尤其对于交互式的应用程序来说简直是无法接受。
（2）第二点主要的缺点，则是这种方式清理出来的空闲内存是不连续的，这点不难理解，我们的死亡对象都是随即的出现在内存的各个角落的，现在把它们清除之后，内存的布局自然会乱七八糟。而为了应付这一点，JVM就不得不维持一个内存的空闲列表，这又是一种开销。而且在分配数组对象的时候，寻找连续的内存空间会不太好找。

复制算法：（新生代的GC）
将原有的内存空间分为两块，每次只使用其中一块
与标记-清除算法相比，复制算法是一种相对高效的回收方法
不适用于存活对象较多的场合，如老年代（复制算法适合做新生代的GC）
复制算法的最大的问题是：空间的浪费
并不需要按照1:1的比例来划分内存空间，而是将内存分为一块比较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor

标记-整理算法：（老年代的GC）：
标记-压缩算法适合用于存活对象较多的场合，如老年代。它在标记-清除算法的基础上做了一些优化。和标记-清除算法一样，标记-压缩算法也首先需要从根节点开始，对所有可达对象做一次标记；但之后，它并不简单的清理未标记的对象，而是将所有的存活对象压缩到内存的一端；之后，清理边界外所有的空间。
标记/整理算法唯一的缺点就是效率也不高。

三个算法都基于根搜索算法去判断一个对象是否应该被回收，而支撑根搜索算法可以正常工作的理论依据，就是语法中变量作用域的相关内容。因此，要想防止内存泄露，最根本的办法就是掌握好变量作用域，而不应该使用C/C++式内存管理方式。
在GC线程开启时，或者说GC过程开始时，它们都要暂停应用程序（stop the world）。
它们的区别如下：（>表示前者要优于后者，=表示两者效果一样）
（1）效率：复制算法>标记/整理算法>标记/清除算法（此处的效率只是简单的对比时间复杂度，实际情况不一定如此）。
（2）内存整齐度：复制算法=标记/整理算法>标记/清除算法。
（3）内存利用率：标记/整理算法=标记/清除算法>复制算法。

## 可触及性：
可触及的：
从根节点可以触及到这个对象。
其实就是从根节点扫描，只要这个对象在引用链中，那就是可触及的。
可复活的：
一旦所有引用被释放，就是可复活状态
因为在finalize()中可能复活该对象
不可触及的：
在finalize()后，可能会进入不可触及状态
不可触及的对象不可能复活
要被回收。


## 泛型
java假泛型：类型擦除
创建泛型类型只能使用反射
```java
if (item instanceof E) { // Compiler error 
    ...
}
E item2 = new E(); // Compiler error 
E[] iArray = new E[10]; // Compiler error 
```

## 字符串全排列
```java
private static void permutation(String perm, String word)
{ 
    if (word.isEmpty()) 
    { System.err.println(perm + word); }
    else 
    { for (int i = 0; i &lt; word.length(); i++) 
        { 
          permutation(perm + word.charAt(i), word.substring(0, i) + word.substring(i + 1, word.length()));
        }
    }
}
```


# 设计模式

## 接口和抽象类
接口用于定义 API，定义了类遵守的规则和行为；提供抽象，同样的定义可以有多种实现；

抽象类可以很好的定义一个类的默认行为，而接口能更好的定义类型，有助于后面实现多态机制

a IS-A hierarchy or CAN-DO-THIS hierarchy

## 原则
单一职责
开闭原则
里氏替换
封装
依赖倒置：易于测试、模拟数据、代码简洁
组合和继承
接口隔离
面向接口编程
委托原则
迪米特法则：
a method M of object O should only call following types of methods:
Methods of Object O itself
Methods of Object passed as an argument
Method of object, which is held in instance variable
Any Object which is created locally in method M

## 适配器和代理模式
适配器模式提供对接口的转换。如果你的客户端使用某些接口，但是你有另外一些接口，你就可以写一个适配去来连接这些接口。

适配器模式用于接口之间的转换，而代理模式则是增加一个额外的中间层，以便支持分配、控制或智能访问。

## 装饰模式
装饰模式的目的是在不修改类的情况下给类增加新的功能



# 高质量代码

## 可读性
1. 不要编写大段的代码，用函数
2. 代码风格统一，名称与注释
3. 流程清晰，细节封装，
4. 简化变量，降低作用域，使用类封装

## 三种组织代码的方法
1. 抽取不相关的子问题
2. 一次只做一件事
3. 把想法变成代码

## 可维护性
1. 灵活性，不能写死
2. 面向接口编程，适度抽象

## 可扩展性
### 策略模式
变继承为组合，使用策略类表示行为


# kotlin 特性

var val 可变 不可变
？ nullable 变量
可选参数，替代方法重载
类扩展，替代将变量作为参数传递
自动get set
out in 泛型
data class 写法更简便
智能转型
if (x is String) {
    print(x.length) // x 被自动转型为String
}
when 更强大的switch

函数式特性：
1. 高阶函数
2. 闭包 lamda

协程
coroutines:
launch
runblock

与java调用：
java调kotlin
在 org.foo.bar 包内的 example.kt 文件中声明的所有的函数和属性，包括扩展函数， 都编译成一个名为 org.foo.bar.ExampleKt 的 Java 类的静态方法。
// example.kt
package demo

class Foo

fun bar() {
}
// Java
new demo.Foo();
demo.ExampleKt.bar();

可以使用 @JvmName 注解修改生成的 Java 类的类名：
@file:JvmName("DemoUtils")
@file:JvmMultifileClass

@JvmField 属性字段暴露
@JvmStatic

kotlin.jvm.JvmClassMappingKt.getKotlinClass(MainView.class)
@JvmOverloads

kotlin调java：
关键字使用转义符`
Java 的装箱原始类型映射到可空的 Kotlin 类型

Java 的通配符转换成类型投影
Foo<? extends Bar> 转换成 Foo<out Bar!>!
Foo<? super Bar> 转换成 Foo<in Bar!>!
Java的原始类型转换成星投影
List 转换成 List<*>!，即 List<out Any?>!

wait()/notify()，类型 Any 的引用不提供这两个方法
可以将其转换为 java.lang.Object：
(foo as java.lang.Object).wait()

getClass() val fooClass = foo::class.java

native，要声明一个在本地（C 或 C++）代码中实现的函数，你需要使用 external 修饰符来标记它：
external fun foo(x: Int): Double

# Groovy特性
现代和动态语言特性
不需申明类型，自动推导
stream api
闭包和lamda
字符串、数组、集合更方便操作


# realm

操作方便，直接操作对象，加上transactions就可以持久化
比sqlite等速度更快，使用java memory mapped

# rxjava

## 函数响应式编程
1. 摆脱回调
2. 方便线程切换
3. 代码简洁

## 几个概念：

Observable
Observer
OnSubscribe

Subscription
Subscriber

Observer 订阅 Observable, 有onCompleted onError onNext三个方法。
Subscriber implements Observer, Subscription，接收Observerable的事件，并且提供手动取消。
OnSubscribe 产生事件的地方，当调用subscribe及有订阅者的时候调用。

## 调用过程
“上游”和“下游”的概念：按照我们写的代码顺序，just 在 map 的上面，Action1 在 map 的下面，数据从 just 传递到 map 再传递到 Action1，所以对于 map 来说，just 就是上游，Action1 就是下游。数据是从上游（Observable）一路传递到下游（Subscriber）的，请求则相反，从下游传递到上游。

调用subscribe请求，表示订阅者准备好接收数据了：
1. 请求过程是从最后的订阅者一直传递到最开始的Observable，请求的过程是构造subscriber的过程，把一个 subscriber 加入到另一个 subscriber 中
2. 接收数据的过程是subscriber执行onNext的过程，每一个subscriber执行完后将数据发送给下游。

# 其他技术问题
## 死锁
互斥条件 不可抢占条件 占有且申请条件 循环等待条件
预防： 打破条件
避免: 
安全序列:是指系统中的所有进程能够按照某一种次序分配资源，并且依次地运行完毕，这种进程序列{P1，P2，...，Pn}就是安全序列
银行家算法是从当前状态出发，逐个按安全序列检查各客户谁能完成其工作，然后假定其完成工作且归还全部贷款，再进而检查下一个能完成工作的客户，
# 网络

## tcp udp
1. 面向连接和无连接
三次握手：syn-synack-ack
四次挥手：fin-ack-fin-ack
2. 可靠性：tcp保证消息送达，如果丢失协议会负责重传
3. 有序性，即使接收的消息无序，协议也会排序
4. data Boundary，tcp不保护数据边界，udp保护。tcp用byte stream传输，udp包单独发送，且只在到达时候检测完整性
5. 速度，tcp慢
6. 头部tcp20byte，udp8byte
7. 拥塞控制，udp没有
8. udp应用：dhcp，dns，

Socket
DatagramSocket

## https
流程
1. client hello
2. server hello 返回ssl证书
3. client 用证书中公钥加密 对称算法和密钥
4. server 确认
5. 请求内容，对称密钥加密
6. 响应内容，对称密钥加密


# scrum

## 是什么
敏捷开发是一种以人为核心、迭代、循序渐进的开发方法
scrum是一个框架，人们可以解决复杂的自适应问题，同时也能高效并有创造性地交付尽可能高价值的产品。
Scrum Master，Product Owner，Development Team

## 基本假设：
1. 外部需求模糊、多变
2. 快速迭代，为此发布时砍功能是正常而不是失败

## scrum master
1. 确保流程的正确
2. 约束官僚主义
3. 保证交叉知识结构

## 如何做：
用户故事-> task -> 开发 -> 测试 ->发布

Sprint计划会议之后:Sprint 计划会议会产生一些实实在在的成果
–sprint目标
–sprint backlog（即 sprint 中包括的故事列表）
–确定好 sprint 演示日期，周期
-把故事拆分成任务，最小可执行单元，任务优先级，核心任务表明
-开发依赖处理：互相依赖的需求同时处理，互不依赖的并行开发；不同分支开发，单独提测，保证主干代码干净
-测试：单独task测试，降低风险
-发布：sprint周期结束，评估完成度，是否可发布

# 数据埋点
1. 功能点埋点，使用率、驻留时间
2. 性能监控：加载时长、上传时长
3. 错误率：功能使用的成功率

# 可读性 结构清晰代码

# 组件化
why
1. 团队负责人不同，开发周期不一致
2. 业务相对独立
3. 无法有效约束代码规范

how：
1. 编译隔离，maven仓库提供aar包，定义版本号规则
2. 定义接口，与主工程交互
3. 基础库lib

优点：
1. 代码隔离、业务模块组件更加独立
2. 公共模块采用依赖库方式，提供给各个业务线使用，减少重复开发和维护工作量
3. 并行开发，提升效率
4. 降低项目熟悉成本和可维护性
5. 编译速度提升

缺点：
1. 对公共库的依赖，出现冲突，第三方升级麻烦
2. 公共业务模块下沉到公共库，导致开发麻烦、公共库变大
3. 各个业务模块之间的接口变化，需要同时协同多人


# 工具
1. 编译时间、大小监控，渠道包；
2. 翻译工具、接口数据测试工具、tinypng工具、10进制到16进制转换，内部技术文档；
3. 


# 其他
项目中遇到最大的困难是什么？如何解决的

自己的优点和缺点是什么？举例说明

你的职业规划以及个人目标；未来发展路线及求职定位
title: android-view-draw-review
tags:
---

# 应用层

1）Measure
深度优先 

2）Layout
深度优先原则递归

3）Draw

# 系统层

SurfaceFlinger的主要工作是：

响应客户端事件，创建Layer与客户端的Surface建立连接。
接收客户端数据及属性，修改Layer属性，如尺寸、颜色、透明度等。
将创建的Layer内容刷新到屏幕上。
维持Layer的序列，并对Layer最终输出做出裁剪计算。

![](https://pic4.zhimg.com/v2-1f7928f598476560d7aae611ee5b6997_b.png)

# 显示刷新机制

## VSYNC
在加入VSYNC信号同步后，每收到VSYNC中断，CPU就开始处理各帧数据。已经解决了刷新不同步的问题。
!(https://pic4.zhimg.com/v2-c94da29521c063bcaea8d227dea1b807_b.jpg)

## Tripple Buffer

当VSYNC时，如果下一帧没有准备好，可以紧接着开始下下一帧。

Tripple Buffer利用CPU/GPU的空闲等待时间提前准备好数据，并不一定会使用


从Android系统的显示原理可以看到，影响绘制的根本原因有以下两方面：

1.绘制任务太重，绘制一帧内容耗时太长。
2.主线程太忙了，导致VSYNC信号来时还没有准备好数据导致丢帧。


实际开发中，我们应该只在主线程做以下几方面的工作：

UI生命周期控制
系统事件处理
消息处理
界面布局
界面绘制
界面刷新
除了这些以外，尽量避免将其它处理放到主线程中，特别是复杂的数据计算和网络请求。


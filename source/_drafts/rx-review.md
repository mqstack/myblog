title: rx review
tags:
---

# 几个概念：

Observable
Observer
OnSubscribe

Subscription
Subscriber

Observer 订阅 Observable, 有onCompleted onError onNext三个方法。
Subscriber implements Observer, Subscription，接收Observerable的事件，并且提供手动取消。
OnSubscribe 产生事件的地方，当调用subscribe及有订阅者的时候调用。

# 调用过程
“上游”和“下游”的概念：按照我们写的代码顺序，just 在 map 的上面，Action1 在 map 的下面，数据从 just 传递到 map 再传递到 Action1，所以对于 map 来说，just 就是上游，Action1 就是下游。数据是从上游（Observable）一路传递到下游（Subscriber）的，请求则相反，从下游传递到上游。

调用subscribe请求，表示订阅者准备好接收数据了：
1. 请求过程是从最后的订阅者一直传递到最开始的Observable，请求的过程是构造subscriber的过程，把一个 subscriber 加入到另一个 subscriber 中
2. 接收数据的过程是subscriber执行onNext的过程，每一个subscriber执行完后将数据发送给下游。


# concatMap and flatMap
What is the difference between concatMap and flatMap

Flat map uses merge operator while concatMap uses concat operator.

As you see the concatMap output sequence is ordered - all of the items emitted by the first Observable being emitted before any of the items emitted by the second Observable,
while flatMap output sequence is merged - the items emitted by the merged Observable may appear in any order, regardless of which source Observable they came from.
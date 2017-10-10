title: Java 和 Kotlin 中的泛型
date: 2015-03-04 11:31:38
categories:
- 编程语言
tags:
- Java
- Kotlin

---
Java中对泛型的引入使得对类、接口和方法的使用有了更多灵活性。而Kotlin作为可运行在jvm之上的一门现代语言，其对泛型的设计又是怎样的呢？本文主要回顾Java的泛型特征并对比Kotlin和Java
<!--more-->

# 什么是泛型

泛型是什么？概念上讲，泛型允许定义类、接口、方法的时候使用参数化类型。

什么叫参数化类型？我的理解是：接口的服务方需要由客户端告诉自己对象的类型，于是将这些类型参数化；而接口的调用方知道自己使用的是什么类型，于是传入相应的类型。

泛型有什么好处呢？

先看下，假如没有泛型，代码会怎么写。以ArrayList为例，ArrayList使用数组存储数据：

```Java
transient Object[] elementData;
```

使用set或add方法时，由于无法指定类型，调用者可以添加任意类型的对象。即使添加了错误类型的对象，也不会产生任何编译期错误。而使用get方法时，调用者只能得到一个Object对象，需要将这个对象再强转为所需的类型。如果List中添加了错误类型的对象，这段代码便会产生运行期错误，显然这是不安全的。如果要想保证类型安全，难道要针对每种类型都将ArrayList实现一遍？这显然是不现实而且愚蠢的。

泛型的使得代码更安全，避免出现运行时错误；同时接口的设计变得更为灵活，不需要针对不同的对象写功能类似的接口。

# Java的泛型

Java 的泛型是从JDK5开始引入，广泛应用于JDK中的集合类框架中。我们从以下几个方面去了解Java的泛型。

## 类型擦除

Java的泛型是只在源码中存在，而在字节码中是不包含类型信息的。举个例子

```java
	List<Integer> list1 = new ArrayList();
	List<String> list2 = new ArrayList();
	System.out.println(list1.getClass() == list2.getClass());
```

这个例子中，看起来list1和list2是两个类型，因为一个只能接收Integer，一个只能接收String。但执行这段代码会打印true，说明泛型在编译的时候被擦除了。字节码中只包含List原始类型了。

因为有类型擦除，Java的泛型有许多奇怪的特性：
1. 泛型类没有自己单独的Class类对象，只存在原始类的对象。
2. 静态变量是被泛型类所有的实例共享的，当然你无法再泛型类中声明一个泛型类型的静态变量。
3. 泛型的类型参数无法用在Java的异常处理的catch中，因为经过类型擦除后，JVM是无法区分两个具有不同泛型类型的类的。比如MyException<String>和MyException<Integer>。

那为什么要这么设计呢？在运行时，就算有泛型，就算能检测出类型错误，也已经为时已晚，没有意义了，除了抛出异常并没有什么有效的挽救措施。而泛型的作用本来就是为了在编译期间做类型检查。经过编译，可以尽可能发现可能出错的地方，当然也不能避免运行时出现类型转换异常的情况。例如，我们可以通过反射调用List的add方法，将String放到List<Integer>中：

```java
	List<Integer> list1 = new ArrayList();
	list1.getClass().getMethod("add", Object.class).invoke(list1, "test");
```

## 通配符与上下界

使用泛型的时候，可以指定具体的类型比如String，也可以用? 通配符来表示类型是未知的，当然也可以使用上下界来表示是某个类型的子类或父类。例如：List<?>, List<? extends Integer>, List<? super Integer>。我们称带有extends的类型是协变的(covariant)，带有super的类型是逆变的(contravariance)。

由于通配符表示未知类型，那传入不匹配的类型便会出现编译错误了。比如：

```java
	List<? extends Integer> list1 = new ArrayList();
	List<? super Integer> list2 = new ArrayList();
	List<? > list3 = new ArrayList();
	Integer i = list1.get(0);
	list1.add(1); //编译错误
	Integer j = list2.get(0); //编译错误
	Object k = list2.get(0);
	list2.add(1);
	Object obj = list3.get(0);
	list3.add(new Object()); //编译错误
```

其中，list1的类型是Integer子类，所以可以读但不能写；list2的类型是Integer的父类，所以可以写但不能读，读出来的类型只能是Object；list3的类型等同于<? extends Object>，因为所有的类都是Object的子类，所以也是可以读但不能写。

通配符如何使用，什么时候使用extends 什么时候使用super呢？可以遵循以下原则：

	producer-extends, consumer-super

如果泛型的变量生产了新的泛型实例，那就使用extends，反过来如果泛型的变量消费了实例，那就使用super。

```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}

public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

上述两个例子很好的说明了生产者和消费者的概念。简单的说，如果一个变量是赋值给E类型的变量，那它的类型可以为<? extends E>；而一个变量可以接收E类型的赋值，那它的类型可以为<? super E>。

另外一个例子，假如有这么一个接口：

```java
interface Source<T> {
  T nextT();
}
```

很明显它其中只有消费的方法，而没有生产的方法，按道理将一个Source<String>变量赋值给Source<Object>是安全的，然而在Java中这却是不允许的。我们只能用Source<? extends Object>声明的变量来使用，这中写法就显得很冗余。

# Kotlin的泛型

在Kotlin中，用了另外一种方式描述接口是消费者还是生产者。方法是给类型参数加上 out 或者 in 的注解。下面这部分参考自官方文档：

## 声明端的可变性 ( declaration-site variance )
所谓的declaration-site，就是指接口或类的定义部分，而不是调用部分。以官方文档上的两个例子说明：

```java
abstract class Source<out T> {
    abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

```java
abstract class Comparable<in T> {
    abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

当使用 out 注解时，表示告诉编译器，这个接口只会返回 T 类型对象而不会接收对象。我们说类 Source 在参数 T 上是协变的，Source 是 T 的生产者而不是消费者。

而使用 in 注解时，表示告诉编译器，这个接口只会接收 T 类型的对象而不会返回。我们说类 Comparable 在参数 T 上是逆变的，Comparable 是 T 的消费者而不是生产者。

## 使用端变异 ( Use-site variance )

有些类既是生产者，又是消费者，那在类的声明中就无法声明它是协变或者逆变了。如下例子：

```java
class Array<T>(val size: Int) {
    fun get(index: Int): T { /* ... */ }
    fun set(index: Int, value: T) { /* ... */ }
}
```

这样类就变得很不灵活了。比如下面的方法，将Array<Int>拷贝至Array<Any>但编译器不允许。

```java
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}

val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 
copy(ints, any) // Error: expects (Array<Any>, Array<Any>)
```

为了不让from的对象被写入，可以加上 out 注解，这样我们就可以正确执行 copy(ints, any) 代码了。

```java
fun copy(from: Array<out Any>, to: Array<Any>) {
 // ...
}
```

同样的，如果要防止对象被读取，可以加上 in 注解。

```java
fun fill(dest: Array<in String>, value: String) {
    // ...
}
```

## 星号推断 ( Star-projections )

星号 * 类似于Java的 ？，表示不知道具体类型，但是更安全。Kotlin的 * 有如下含义：

- 对于Foo<out T>，其中 T 是上限 TUpper 的协变类型。Foo<*> 等同于 Foo<out TUpper>，表示 T 类型未知时，可以安全的读取为 TUpper。
- 对于Foo<in T>， 其中 T 是逆变类型。Foo<*>等同于Foo<in Nothing>，表示 T 类型未知时，不能写入。
- 对于Foo<T>， 其中 T 是不可变类型。Foo<*>在读取时等同于Foo<out TUpper>，写入时等同于Foo<in Nothing>。


总体来看，由于需要编译成class文件运行与jvm中，所以Kotlin 的泛型整体概念与 Java 几乎相同。但是语法层面更为强大，对于可变性的声明也更为简洁。

# 参考

http://www.infoq.com/cn/articles/cf-java-generics

http://kotlinlang.org/docs/reference/generics.html
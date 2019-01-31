---
layout: post
title: Kotlin 的使用
---
# Kotlin 的初次使用
----

最近由于项目需要，开始学习 Kotlin 来写 Android，Kotlin 已经火了很长的一段时间了，但是之前我也就知道 fun 是函数这样的语法，这次有机会能好好学习一下 Kotlin 了。没有写很多，有这么几点想法吧。

先说几个优点。

###Kotlin 在 Android 的适配

Kotlin 在 Android 里适配特别好，能实现直接的混写，如果想使用 Kotlin， 可以直接在现有的项目中使用，完全可以老代码用 java， 新代码用 Kotlin 的形式来写。而且 IDE 提供 java 代码自动转换 Kotlin，从老文件拷代码过来会帮你自动转好，都很方便。现在遇到了一点小坑，Kotlin 有强制变量不为空的申明，在函数形参调用时就会检查，如果为空就会直接报错，这真的很坑啊，java 那边没有这个，w很多时候，java代码转到 kotlin 时，编译器检查不出来，但是会运行错误。除了这个之外，还没遇到别的混写不好的地方。

###Kotlin 的函数式列表操作

好用的列表函数式操作，这个我觉得是 Kotlin 能提高开发效率的最主要的一点

```kotlin
list.map { it.name } //将 list 中所有元素的 name 组成一个新的元素
list.find { it.id == id} //找到 list 中元素等于 id 的 
list.filter { it.source.contain(target) } //筛选 source 中含有 target 的元素
```

就是类似这样的一些用法，以前在 java 里需要写一个for循环的，大多可以用这样的函数式操作完成。

### 好用的语法糖

Kotlin 有很多的语法糖，这些都能让开发效率变得高一些，在第一次接触时会感到一些困惑，但是等熟悉之后，用起来就很顺手了。

1. getter 和 setter 的封装，如果你把一个类的成员用 getter 和 setter 封装好，那么在别的地方调用时，就可以直接使用那个对象的成员了。

2. 比 java 更美观的 lambda 表达式

   ```java
   view.setOnClickListener(view -> {
       method();
   });
   view.setOnClickListener(this::method); //java 的两种匿名类简化写法
   ```

   ```kotlin
   view.setOnClickListener { method()} // Kotlin lambda 写法
   ```

   这两种写法，还是 Kotlin 写的更自然，java 方法应用 第一次见的人还会蒙圈，需要反应半天才能意识过来。关于 java 和 Kotlin 的 lambda 表达式写法，以后总结一下，代码整洁很有帮助。

3. 类型解析

   Kotlin 和 java 都是基于 JVM，所以大家在字节码上跑的都是一样的，所以Kotlin 和 java 一样，都是静态类型的语言，即在编译期，变量的类型就会被定义好。但是 Kotlin 有自动的类型解析，每次新建变量的时候会根据赋值的类型，判断变量的类型，就有了如下的代码，看起来真的很像动态类型。

   ```kotlin
   var view = getViewByID(R.id.textview) as TextView
   val str = "Hello word!"
   view.setText(str)
   ```

   完全没有看到变量的类型定义，其实这都是通过编译器推断出来的，`view`变量在第一次赋值后，便不能再赋值成别的类型了，说明 Kotlin 还是静态语言。

### Kotlin 协程

Kotlin 中对异步操作的改进，这部分我还没有太深入的了解过，只看到 协程是非常轻量级的，而且也好用的一个异步机制。其实在 Android 里通常的异步操作有 网络请求和 IO 操作，图像处理等，我现在的项目里，IO操作，图像处理都比较少，所以用到的地方不会很多。看了网上的介绍 协程要比 RxJava 要好用，RxJava 我也没用过啊，我觉得 Android，java 已经有很多异步操作的工具了，Handler，AsyncTask，ThreadPool，(这些还不够你们用吗？) 内心感觉现在这些是有点多余的。

不过未来计算机的前景是多核心，并发计算才是以后未来，也许有那么一天，大部分的运行都可以并行计算，那时候，异步也就是必须要有的操作了



说完了优点，可以说一下 Kotlin 的缺点了，Kotlin虽然帮我解决了一下 Java 中的问题，但是也带来了一些新的问题，综合考虑下来，基本上是 五五开，所以Kotlin 也没有带来那么大的好处。

###Kotlin 的空检查

给大家讲个笑话，Kotlin 的空对象检查。

Kotlin 为了避免 null 的发生，就添加了非空对象这么个机制，而且对一切对象一视同仁，包括基础变量，就像这个`Int!`。默认情况下，你定义的变量都是带感叹号的，非空变量，也就是说它只能接受非空的赋值。或者你也可以加个问号，这样它就不会接受 空检查了，

其实这个特性也有一定的好处，提醒你进行空检查，但是有时候，代码是多个人写的，比如你夹在中间，上面一层给你 `?`  ，下面一层问你要`!`。这个时候就很难受了。虽然说空处理总是要进行的吧。但是我觉得空处理完全可以作为惰性处理， 出现问题之后再去解决， java 解决 null 异常应该算是最简单的了。

保证非空的变量的灵活性很低，有时候必须用运行空的变量，这时候就和 java 一样。总的下来说，这个特性的好坏参半。

### Kotlin的新语法

Kotlin 的新语法有点太多了，关键字和特殊用法都很多，`object` `companion` `?` `!!` `var` `val` 

好吧，好像也没有那么多，就是和 Java 用惯了之后，切到 Kotlin 上刚开始会觉得陌生，熟悉了之后也还可以。但是和 Kotlin 带来的好处相比，我觉得差不多 五五开吧。

以上




---
layout: post
title: Kotlin 协程在 Android 的应用
---
# Kotlin 协程在 Android 中的应用
----

> Kotlin 的 *挂起函数* 概念为异步操作提供了比 future 与 promise 更安全、更不易出错的抽象。

### Future 与 Promise 模式的介绍

在计算机科学中，future、promise、delay和deferred是指用于在某些并发编程语言中同步程序执行的构造。由于某些计算（或者网络请求）尚未结束，我们需要一个对象来代理这个未知的结果，于是就有了上述这些构造（future、promise等）

```
 t1 := x <- a();
 t2 := y <- b();
 t3 := t1 <- c(t2);
```

其中`x <- a()`表示将消息a()异步发送给`x`。所有三个变量都会立即为其结果分配future，执行过程将继续进行到后面的语句。之后尝试解决t3的值可能会导致延迟；但是，流水线操作可以减少所需的往返次数。如果与前面的示例一样，x、y、t1和t2都位于相同的远程机器上，则流水线实现可以用一次往返来计算`t3`，不必用三次。由于所有三条消息都指向同一远程计算机上的对象，因此只需要发送一个请求，只需要接收一个包含结果的响应。另请注意，即使t1和t2位于不同机器上，或者位于与x或y不同的机器上，发送`t1 <- c(t2)`也不会阻塞。(wiki百科)


### 协程进行并行计算


coroutine 协同程序

使用 Kotlin 协程前需要 依赖 kotlinx-coroutines-core 

Maven:

```xml
<!-- https://mvnrepository.com/artifact/org.jetbrains.kotlinx/kotlinx-coroutines-core --> 
<dependency> 
   <groupId>org.jetbrains.kotlinx</groupId> 
   <artifactId>kotlinx-coroutines-core</artifactId> 
   <version>1.1.1</version> 
</dependency> 
```

Gradle:

```
//https://mvnrepository.com/artifact/org.jetbrains.kotlinx/kotlinx-coroutines-core 

compile group: 'org.jetbrains.kotlinx', name: 'kotlinx-coroutines-core', version: '1.1.1' 
```



1. runBlocking 用以创建一个阻塞的协程

2. GlobalScope.launch {} 用以创建一个新的 Job 新的协程，非阻塞 这也就是挂起函数

3. Job.join() 将会阻塞当前协程，直到 job 任务完成

4. 如果 job 是在 runBlocking 里launch的协程，那么 会自动的在末尾join该协程

5. Job.cancel 会将当前的任务主动取消 job.cancelAndJoin() 是等待结束并join

6. finally 会在协程被取消时运行

7. withContext(NonCancellable) 代码块里可以运行不能取消的协程

```kotlin
val job = launch {
    withContext(NonCancellable) {
            repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
}
```

这个job 就不会被 cancel 调

8. withTimeout(1300L) 代码块里可以确保不会超时，超时自动退出

```
fun main() = runBlocking {
    withTimeout(1300L) {
        repeat(1000) { i ->
                println("I'm sleeping $i ...")
            delay(500L)
        }
    }
}
```

9. 通过 async 实现 future promise 模型

```kotlin
fun main() = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")    
}
```



10. coroutineScope 是一个协程作用域 当他的内部发生异常后，内部的所有协程都会停止运行

```kotlin
coroutineScope {
    //doSomething
}
```


11. 调度器与线程。协程上下文包括了一个 协程调度器（CoroutineDispatcher），它确定了相应的协程在执行时使用一个或多个线程。协程调度器可以将协程的执行局限在指定的线程中，调度它运行在线程池中或让它不受限的运行。

```kotlin
launch { // 运行在父协程的上下文中，即 runBlocking 主协程
    println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
}
launch(Dispatchers.Unconfined) { // 不受限的——将工作在主线程中
    println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
}
launch(Dispatchers.Default) { // 将会获取默认调度器
    println("Default               : I'm working in thread ${Thread.currentThread().name}")
}
launch(newSingleThreadContext("MyOwnThread")) { // 将使它获得一个新的线程
    println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
}
```

12. 协程是一种树状结构，协程里可以有子协程，当父协程会等待子协程结束后才会退出，当父协程被取消时，他全部的子协程也将取消



### 协程 Android 实战


1. 将 Activity 实现CoroutineScope接口

```kotlin
class QuestionTagManageActivity : BaseRecyclerViewActivity<BaseData>(), CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main  //Main 为UI 线程，这样 启动的协程中，都是默认以 Main 为主要线程的
}
```



2. 网络请求改写, 通过调用 syncCall 方法，同步调用网络请求，举个列子 

```kotlin
fun getQuestionTagList() = GlobalScope.async(Dispatchers.IO) {
    mGetTagListApi?.cancel()
    mGetTagListApi = GetQuestionTagListApi()
    SolarRequest(mGetTagListApi).syncCall(activity)  //同步调用接口
}

    override fun onRefresh() {
        launch {//通过launch 开一个不阻塞的协程
            try {
                //该协程在 UI 线程中，但是阻塞多久都不会 ANR
                val result = getQuestionTagList().await()
                if (result.success) {
                    //在这里处理业务逻辑
                    val list = result.result as List<QuestionTagVO>
                    updateAdapterDatas(list)
                    tagList.clear()
                    tagList.addAll(list)
                } else {
                    throw HttpException("data")
                }
            } catch (a: Exception) {
                if (datas.size == 1 && datas[0] is StateData && (datas[0] as StateData).state == StateData.StateViewState.loading) {
                    (datas[0] as StateData).state = StateData.StateViewState.failed
                    adapter.notifyDataSetChanged()
                }
            }
        }
    }
```

3. 需要多线程计算的地方，比如加载大图，以前可以通过线程池来加载，现在可以通过使用协程，当然，加载图片的时候换 IO 进程去使用。

下面这块代码可以在 activity 环境中使用，而且不会 block 主线程

```kotlin
launch(Dispatchers.Main) {
    val image = withContext(Dispatchers.IO) { getImage() } // Get from IO context
    imageView.setImageBitmap(image) // Back on main thread
}
```


### kotlin 通道


通道给我的感觉和 Android Handler 是很像的，都是 生产者-消费者模型，而且 handler 和 Chanel 都支持挂起的操作，也就是 handler 就算没有消息，对于线程是没有影响的，chanel 也是。不过 chanel 应该比 handler 要更加灵活，比如消息的接受与发送方，完全取决于 chanel 的处于的协程，而且消息的分发也是公平的，handler 的话是有一套消息分发的规则，如果有 runable 就在对应的线程里去跑，没有runable就是谁发送谁处理。写一个 list 来对比一下吧

|                  | Chanel                           | Handler                                                     |
| ---------------- |:--------------------------------:|:-----------------------------------------------------------:|
| 用途             | 传递协程之间的消息               | 传递线程之间的消息                                          |
| 生产者           | 任意一个协程调用 Chanel 的 send  | 可以在任意一个线程中调用 Handler post                       |
| 生产元素         | 任何对象，包括lambda 表达式      | 序列化对象                                                  |
| 消费者           | 任何协程中调用 Chanel 的 receive | Handler 自身，如果生产元素是 runable，在 handler 线程中消费 |
| 能否实现pipeline | 可以                             | 比较复杂                                                    |
| 挂起             | 支持                             | 支持                                                        |

这么看来，Chanel 可以代替 Handler 实现 跨线程的操作，毕竟协程是随便跨线程的。但是有些 view.post(Runable) 的用法，是Android 默认支持的，当然是不能替代

- 下面展示 Channel 的基本用法

```
fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
        for (x in 1..5) channel.send(x * x)
    }
    // 这里我们打印了 5 次被接收的整数：
    repeat(5) { println(channel.receive()) }
    println("Done!")
}
```

- 发现一个 bug，运行下面这段程序似乎会死锁， withTimeout 和 channel 之间有矛盾

```
fun main() = runBlocking {
    val channel = Channel<Int>()
    launch {
        withTimeout(1000L) {
            for (x in 1..10) {
                println("send ${x * x}")
                channel.send(x * x)
            }
            channel.close()
        }
    }
    delay(1500L)
    for (y in channel) println(y)
    println("Done!")
}
```

- Channel 可以send lambda 函数

```kotlin
fun main() = runBlocking {
    var cur = produce {
        send{x:Int, y:Int -> x+y}
        send{x:Int, y:Int -> x-y}
        send{x:Int, y:Int -> x*y}
        send{x:Int, y:Int -> x/y}
    }
    runState(cur)
    coroutineContext.cancelChildren() // 取消所有的子协程来让主协程结束
}

suspend fun CoroutineScope.runState(method: ReceiveChannel<(Int, Int)-> Int>){
    for(m in method){
        println(m(10,2))
    }
}
```

- Channel 还可以写成一种 pipeline 的形式

```
fun main() = runBlocking {
    val numbers = produceNumbers() // 从 1 开始生产整数
    val squares = square(numbers) // 对整数做平方
    for (i in 1..5) println(squares.receive()) // 打印前 5 个数字
    println("Done!") // 我们的操作已经结束了
    coroutineContext.cancelChildren() // 取消子协程
}

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // 从 1 开始的无限的整数流
}

fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
```


### 总结


Kotlin 协程真的很好用啊：

- 它很轻量级，随便 launch 一个协程都不用担心开销问题。
- 它支持挂起，这点也不用太担心死锁的问题了，只要你开一个非阻塞的 协程去等待结果就好了
- 它可以随意的切换线程，对于Android的开发真的有用啊，以前一般通过 向Main thread post runable 去设置 UI 现在可以直接在 Main thread 里建立一个协程去修改 UI。
- 它还可以通过 channel 在协程之间传递值，任何值，还是很厉害的

Kotlin 还是一门很年轻的语言，bug 什么的还是挺多的，希望Kotlin能好好发展，以后成为一种静态，动态混合语言，兼顾灵活性和安全性强大语言。


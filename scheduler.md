## scheduler

##### 前置知识

**记住，对于 OS scheduler 来说，最重要的是不要让一个 CPU 核心闲着，尽量让每个 CPU 核心都有任务可做。**

> OS scheduler 保证如果有可以执行的线程时，就不会让 CPU 闲着。
>
> 并且它还要保证，所有可执行的线程都看起来在同时执行。
>
> 另外，OS scheduler 在保证高优先级的线程执行机会大于低优先级线程的同时，不能让低优先级的线程始终得不到执行的机会。
>
> OS scheduler 还需要做到迅速决策，以降低延时。

#### goroutine

> 什么是goroutine？

Goroutine 可以看作对 thread 加的一层抽象，它更轻量级，可以单独执行。因为有了这层抽象，==Gopher (程序员)不会直接面对 thread==，我们只会看到代码里满天飞的 goroutine。操作系统却相反，管你什么 goroutine，我才没空理会。我安心地执行线程就可以了，线程才是我调度的基本单位。

> goroutine VS Thread

- **内存占用**

创建一个 goroutine 的栈内存消耗为 2 KB，实际运行过程中，如果栈空间不够用，会自动进行扩容。创建一个 thread 则需要消耗 1 MB 栈内存，而且还需要一个被称为 “a guard page” 的区域用于和其他 thread 的栈空间进行隔离。

对于一个用 Go 构建的 HTTP Server 而言，对到来的每个请求，创建一个 goroutine 用来处理是非常轻松的一件事。而如果用一个使用线程作为并发原语的语言构建的服务，例如 Java 来说，每个请求对应一个线程则太浪费资源了，很快就会出 OOM 错误（OutOfMermoryError）。

- **创建和销毀**

==Thread== 创建和销毀都会有巨大的消耗，因为要和操作系统打交道，是==内核级==的，通常解决的办法就是线程池。而 ==goroutine== 因为是由 Go runtime 负责管理的，创建和销毁的消耗非常小，是==用户级==。

- **切换**

当 threads 切换时，需要保存各种寄存器，以便将来恢复，而 goroutines 切换只需保存三个寄存器：Program Counter, Stack Pointer and BP。

一般而言，线程切换会消耗 ==1000-1500 ns==，一个纳秒平均可以执行 12-18 条指令。

Goroutine 的切换约为 ==200 ns==，相当于 2400-3600 条指令

> M:N 模型

我们都知道，==Go runtime==会负责 goroutine 的生老病死，从创建到销毁，都一手包办。==Runtime 会在程序启动的时候==，创建 ==M 个线程==（CPU 执行调度的单位），之后创建的==N 个 goroutine 都会依附在这 M 个线程上==执行。这就是 M:N 模型。

在同一时刻，一个线程上只能跑一个 goroutine。

#### 初了解schedule

> schedule是什么？

用户程序进行的系统调用都会被 Runtime 拦截，以此来帮助它进行调度以及垃圾回收相关的工作。

Go scheduler 是 Go runtime 的一部分，它内嵌在 Go 程序里，和 Go 程序一起运行

> 为什么要用schedule？

Go scheduler 可以说是 Go 运行时的一个最重要的部分了。Runtime 维护所有的 goroutines，并==通过 scheduler 来进行调度==。Goroutines 和 threads 是独立的，但是 goroutines 要依赖 threads 才能执行。

Go 程序执行的高效和 scheduler 的调度是分不开的。



#### schedule底层原理

**G\*M\*P**

- ==P==代表控制器，调度G到M上，其维护了一个队列，存储了所有需要它来调度的G。
- ==G==代表一个go routine单元
- ==M==代表内核线程，所有的G都要放在M上才能运行。

go程序启动时，会给==每个逻辑核心分配一个P==，同时为每个P（Logical Processor）分配一个M（Machine，表示内核线程）。内核线程在cpu核心上调度，而G 则是在 M 上进行调度。（**m的数量是不可控**的 ==m的数量>=p的数量== m的数量在运行时 动态变化）

Go scheduler 的职责就是将所有处于 runnable 的 goroutines 均匀分布到在 P 上运行的 M。（**将G平均分配到P上运行的M**）

**两个重要组件**

- 全局可运行队列（**GRQ**），LRQ 存储本地（也就是具体的 P）的可运行 goroutine
- 本地可运行队列（**LRQ**），GRQ 存储全局的可运行 goroutine，这些 goroutine 还没有分配到具体的 P。

<img src="C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210205214643098.png" alt="image-20210205214643098" style="zoom:80%;" />

#### goroutine 调度时机

在四种情形下，goroutine 可能会发生调度，但也并不一定会发生，只是说 Go scheduler 有机会进行调度。

| 情形            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| 使用关键字 `go` | go 创建一个==新的 goroutine==，Go scheduler 会考虑调度       |
| GC              | 由于进行 GC 的 goroutine 也需要在 M 上运行，因此肯定会发生调度。当然，Go scheduler 还会做很多其他的调度，例如调度不涉及堆访问的 goroutine 来运行。GC 不管栈上的内存，只会回收堆上的内存 |
| 系统调用        | 当 goroutine ==进行系统调用时，会阻塞 M==，所以它会被调度走，同时一个新的 goroutine 会被调度上来 |
| 内存同步访问    | ==atomic，mutex，channel 操作等会使 goroutine 阻塞==，因此会被调度走。等条件满足后（例如其他 goroutine 解锁了）还会被调度上来继续运行 |

#### 当M阻塞了怎么办

1、**同步下条件**

- 当一个OS线程M由于io操作而陷入阻塞，假设此时G0正跑在了M上，那么M上绑定的P就会==带着余下的所有G去寻找新的M==。
- 当M恢复过来时，一般情况下，会==从别的M上拿过来一个P==，并把原先跑在其上的G0放到P的队列中，从而运行G0。如果，==没有拿到可用的P的话，就把G0放入到全局global runqueue队列==中，使G0等待被调度，然后M进入线程缓存。
- 所有的P也会==周期性的检查global runqueue==并运行其中的goroutine，否则global runqueue上的goroutine永远无法执行。

2、**异步条件下**

- 对于异步的情况，M 不会被阻塞，G 的异步请求会被“代理人” network poller 接手，G 也会被绑定到 network poller。
- 等到系统调用结束，G 才会重新回到 P 上。M 由于没被阻塞，它因此可以继续执行 LRQ 里的其他 G。

![image-20210205220743500](C:\Users\Cristiano-Ronaldo\AppData\Roaming\Typora\typora-user-images\image-20210205220743500.png)


















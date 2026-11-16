---
title: "Java 后端面试：Java 基础与并发"
description: "集合、并发、线程池、锁与线程安全；共 18 道复习题。"
date: "2026-11-15"
category: "Java 后端"
javaDirection: "fundamentals"
tags: ["Java", "面试准备"]
---

> **版本说明：** 基础实现以 JDK 8 为主要示例；涉及 JDK 21/24/25 等新版时会单独注明。不同实现与业务运行版本可能不同。

[← 返回 Java 后端题目总目录](../java-backend-interview-94/)
<span id="1-java-基础与并发"></span>

## Java 基础与并发

### 1 ArrayList 和 LinkedList 如何选择

<span class="java-label java-insight">理解与追问：</span> 例如频繁按列表下标随机读取时，优先考虑数组连续存储；如果要按用户 ID 查找，还需要先建立 ID 到下标或对象的索引；如果操作主要是两端入队出队，可比较 ArrayDeque。若需要在中间删除，即使链表修改链接很快，也要计入找到节点的遍历成本。

ArrayList 基于可扩容数组,按下标访问为 O(1),尾部追加均摊 O(1),中间插入、删除通常要移动元素,为 O(n)。LinkedList 基于双向链表,按下标定位为 O(n);只有已经定位到节点或操作首尾时,链接调整才是 O(1)。选择要结合访问方式和数据规模。

- ArrayList 扩容会分配新数组并复制元素;已知数量时可合理预估容量。

- LinkedList 每个节点还保存前后引用,节点分散;不能只按插入、删除次数判断性能。

- 两者都不保证线程安全;队列、栈场景也应考虑 ArrayDeque。

<span class="java-label java-warning">易错提醒：</span> “LinkedList 任意位置插入、删除都是 O(1)”漏算了定位节点的成本。

<span class="java-label java-example">具体场景：</span> 假设订单列表有 10 万条，页面频繁读取第 5 万条：ArrayList 可通过数组下标定位；LinkedList 需沿节点遍历。若只做队列两端的添加和移除，应优先比较 ArrayDeque，而不是因为“有删除”就选链表。

**追问：链表插入不是 O(1) 吗？** 只有已持有目标节点或操作端点时，调整链接才是 O(1)；从下标查找节点仍是 O(n)。实际选择还要考虑数组局部性、对象分配和数据规模，用代表性负载测量。

### 2 如何解决 Hash 冲突 HashMap 怎样处理

<span class="java-label java-insight">理解与追问：</span> 哈希表先用 hash 定位候选桶，再用 equals 确认具体键。比如两个不同订单号碰到同一个桶不会互相覆盖；只有键被判定相等时，put 才会替换原值。

不同键可能映射到同一桶。常见处理方法有链地址法、开放定址法、再哈希法等。JDK 8 及后续常见版本的 HashMap 使用数组加链表,并在条件满足时转为红黑树;定位桶后,仍需比较键是否相等,不能只比较哈希值。

- hashCode 相同不代表 equals 相等;equals 相等的对象必须具有相同 hashCode。

- JDK 8 的树化相关常量为阈值 8、最小数组容量 64;数组较小时通常先扩容,不能只背节点数量。

- 作为键使用期间,不应修改参与 hashCode、equals 的字段,否则可能再也定位不到原条目。

<span class="java-label java-warning">易错提醒：</span> 扩容能改善桶分布,但不能保证消除所有哈希冲突。

<span class="java-label java-example">具体场景：</span> 假设两个不相等的订单 ID 算出相同的桶下标，它们可以共存于同一桶；查找时再用 equals 找到对应键。若把已放入 Map 的键修改后，其 hashCode 改变，后续查询可能跑到另一个桶而找不到。

**追问：哈希相同就一定是同一个键吗？** 不一定。哈希用于缩小查找范围，最终还要用相等语义判断。若两个对象 equals 为 true 却返回不同 hashCode，就可能被放入不同桶，破坏映射的查找预期。

### 3 HashMap 在多线程下会出现哪些问题

<span class="java-label java-insight">理解与追问：</span> 举例：两个线程同时给同一映射执行 get 后加一再 put，即使换成 ConcurrentHashMap，也可能丢失一次加法；应使用 merge 或 compute 把该键的更新合并为原子操作。

HashMap 没有提供并发访问保护。多个线程并发修改时,可能丢失更新、覆盖条目、计数不准确,读线程也缺少可靠的可见性保证。JDK 7 并发扩容中的链表成环是历史问题;JDK 8 改变了实现,仍不能据此认为线程安全。

- 迭代器的 fail-fast 只是尽力发现修改,不能依赖 ConcurrentModificationException 保证正确性。

- 共享只读的 HashMap 也要安全发布;写入场景可用 ConcurrentHashMap 或统一外部加锁。

- ConcurrentHashMap 的单个操作有并发保证,但 get 后再 put 仍是复合操作,应使用 compute、merge 等适合的原子接口。

<span class="java-label java-warning">易错提醒：</span> modCount 是修改检测计数,不是同步机制。

<span class="java-label java-example">具体场景：</span> 初始计数是 0，线程 A、B 同时读到 0，各自计算出 1 后写回，最终仍是 1。即使容器改为 ConcurrentHashMap，`get` 和 `put` 分两步仍可能发生；可用 `merge(key, 1, Integer::sum)` 原子更新单个键。

**追问：换成 ConcurrentHashMap 就解决所有并发问题了吗？** 不能。它提供单次操作的并发保证，先 get 再 put 仍可能丢更新；例如计数可考虑 merge，但多个键之间的业务约束仍需要额外同步或事务。

### 4 volatile 和 synchronized 分别保证什么

<span class="java-label java-insight">理解与追问：</span> 可见性回答的是一个线程的写能否被另一个线程看到；互斥回答的是多个线程能否同时进入临界区。用 volatile 做停止标记可以，但库存检查和扣减这样的复合操作仍需要锁或原子方案。

volatile 为变量读写提供可见性和内存顺序保证:对某个 volatile 变量的写,happens-before 后续对它的读。单次 volatile 读写具有原子性,包括 long、double;但 i++ 这种读、改、写复合操作并不原子。synchronized 则通过同一监视器互斥保护临界区,并建立解锁到后续加锁的可见性关系。

- 所有相关访问必须遵守同一同步规则,才能保护业务不变量;两个不同锁不能互相保护。

- volatile 引用不等于对象内部所有字段和操作都自动线程安全。

- synchronized 保证互斥,不表示线程不能被调度、中断请求不能发生,也不表示区内禁止一切重排。

<span class="java-label java-warning">易错提醒：</span> 不要笼统地说“volatile 完全不保证原子性”或“synchronized 让线程不能被打断”。

<span class="java-label java-example">具体场景：</span> 一个线程执行 `count++` 时可能先读取 count、加一再写回；两个线程都读到 0，最终可能只写入 1。把 count 标记 volatile 不能把三步合成一步；用 AtomicInteger 的 incrementAndGet 或在同一把锁下更新。

**追问：volatile 为什么不能保证 i++？** 它让每次读写具备相应可见性和顺序，但 i++ 由读取、计算、写回组成，两个线程可能读到同一个旧值。可用 AtomicInteger 的原子更新或统一加锁保护复合操作。

### 5 Condition 与 Object.wait notify 有什么区别

<span class="java-label java-insight">理解与追问：</span> 生产者和消费者可以使用同一把锁下的 notFull、notEmpty 两个条件队列。等待时用 while 重查条件：被唤醒只表示有机会再次检查，并不代表条件仍然成立。

Object 的 wait、notify、notifyAll 与对象监视器配合使用;Condition 的 await、signal、signalAll 与对应 Lock 配合使用。等待会原子地释放关联锁,返回前重新获取锁。一个 Lock 可以创建多个Condition,将不同条件的等待者分组通知。

- 使用 ReentrantLock 的 Condition 时,await 和 signal 都应在持有对应锁时调用,并在 finally 中解锁。

- 条件判断必须使用 while 循环,处理虚假唤醒以及唤醒后条件再次变化。

- signal 选择该条件队列的等待者,不代表可按线程 ID 精确唤醒;被通知后仍需竞争或重新获取锁。

<span class="java-label java-warning">易错提醒：</span> notify、signal 不会立即释放当前线程持有的锁。

<span class="java-label java-example">具体场景：</span> 容量为 10 的队列已满，生产者持锁检查后调用 notFull.await，释放锁并等待；消费者取走元素后持锁 signal。生产者被唤醒后先重新获取锁，再用 while 检查是否仍有空位。

**追问：为什么等待条件要写 while？** 线程可能虚假唤醒，也可能唤醒后重新拿到锁时条件已被其他线程改变。while 在每次恢复后重新检查业务条件；signal 只是通知，并不会立即交出锁。

### 6 sleep yield join interrupt 分别做什么

<span class="java-label java-insight">理解与追问：</span> 当任务收到中断时，需要检查中断或响应阻塞方法抛出的异常并清理资源。不能把 interrupt 当成强制终止；持有的锁也要通过 finally 正确释放。

这四个方法的作用不同：

- **`sleep`**：暂停当前线程一段时间，不释放它持有的监视器。
- **`yield`**：提示调度器当前线程愿意让出执行机会，该提示可能被忽略。
- **`t.join()`**：让调用线程等待线程 `t` 终止。
- **`interrupt`**：发出协作式中断请求；具体反应取决于线程正在执行什么，以及是否检查中断。

- sleep、join、wait 等可中断等待抛出 InterruptedException 时会清除中断标志;无法继续处理时通常应恢复标志或向上传播。

- start 才会启动新线程;直接调用 run 通常只是当前线程执行普通方法。

- await 属于 Condition、CountDownLatch 等工具,不是 Thread 的通用方法。

<span class="java-label java-warning">易错提醒：</span> yield 不保证其他线程马上运行;interrupt 也不等于强制杀死线程。

<span class="java-label java-example">具体场景：</span> 线程池任务在 sleep 时收到 interrupt，sleep 抛 InterruptedException 并清除中断标志。如果当前层无法决定停止策略，可清理资源后 `Thread.currentThread().interrupt()`，再退出或让上层处理。

**追问：中断后任务一定停止吗？** 不一定。interrupt 是协作信号；阻塞方法可能抛异常，计算循环则需要检查标志。捕获 InterruptedException 后若无法处理，通常恢复中断标志或继续向上抛出。

### 7 线程安全的单例有哪些实现

<span class="java-label java-insight">理解与追问：</span> 如果对象初始化代价很高且未必使用，静态内部类可延迟初始化；如果构造简单，饿汉式更容易维护。无论哪种实现，都要单独考虑对象内部共享状态的并发控制。

常见实现包括静态成员饿汉式、静态内部类、枚举和双重检查锁。饿汉式与静态内部类利用类初始化机制保证初始化安全;静态内部类还能延迟到使用 Holder 时初始化。双重检查锁要求实例字段为 volatile,并在同步块内再次检查是否为空。

- 双重检查中的 volatile 用于安全发布和必要的内存顺序,避免其他线程观察到未完成初始化的对象。

- 枚举实例具有语言层面对枚举构造和序列化的约束;普通类若参与序列化,还应考虑 readResolve。

- 单例通常只在定义它的类加载器范围内唯一;实例唯一不代表实例中的可变业务状态线程安全。

<span class="java-label java-warning">易错提醒：</span> 只在 if(instance == null) 后直接 new,不能保证并发下只创建一次。

<span class="java-label java-example">具体场景：</span> 双重检查中，两个线程同时发现 instance 为空，一个先进入锁并创建；第二个获得锁后必须再次检查，避免再创建一次。instance 使用 volatile 保障引用安全发布；实例里的可变字段仍需单独处理并发。

**追问：双重检查为什么要 volatile？** 它为实例引用的发布提供必要的可见性和顺序约束，防止其他线程观察到未安全初始化的实例。还要在锁内再次检查，否则多个等待线程仍可能重复创建。

### 8 常见线程池与 ThreadPoolExecutor 七个参数是什么

**从提交任务看七个参数如何配合：** 假设核心线程数为 2、最大线程数为 4、队列容量为 3，且已有任务均未完成：任务 1、2 分别创建核心线程；任务 3、4、5 进入队列；任务 6、7 因队列已满而触发新建线程，线程数达到 4；任务 8 仍无法入队且不能再扩容，于是执行拒绝处理器。`keepAliveTime` 与 `unit` 控制多余空闲线程的存活时间，`threadFactory` 决定如何创建线程。

这只是说明默认接收流程：任务一旦结束、线程退出或队列状态发生变化，实际走向也会变化；核心线程超时等配置同样会影响回收。更多边界可查 [JDK 8 ThreadPoolExecutor 文档](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html)。

<span class="java-label java-insight">理解与追问：</span> 例如核心线程 4、最大线程 8、队列容量 100：先创建核心线程，再尝试排队；队列满后才可能扩容，达到上限仍无法接收时执行拒绝策略。这里要结合实际任务持续时间观察队列等待。

常见工厂方法有固定线程数、单线程、缓存线程池和定时线程池。自定义 ThreadPoolExecutor 的七个参数是 corePoolSize、maximumPoolSize、keepAliveTime、unit、workQueue、threadFactory、handler。正常接收任务时通常先补足核心线程,再入队;队列不能接收时再扩容到最大线程数,仍不能接收才拒绝。

- newFixedThreadPool、newSingleThreadExecutor 通常使用无界队列,持续过载可能造成任务堆积。

- newCachedThreadPool 使用 SynchronousQueue 直接交接任务,最大线程数很大,持续过载可能造成线程膨胀;定时任务用 ScheduledThreadPoolExecutor。

- keepAliveTime 默认作用于超过核心数量的空闲线程,也可配置核心线程超时;线程工厂可设置业务可识别的名字。

- 队列容量、线程数应结合耗时、阻塞比例、下游容量和压测数据决定,不存在通用最优值。

<span class="java-label java-warning">易错提醒：</span> 不存在 new ThreadExcutors“默认线程池”;无界队列下只调大 maximumPoolSize 通常不起作用。

<span class="java-label java-example">具体场景：</span> 核心线程 4、最大线程 8、队列容量 100：前 4 个并发任务占用核心线程；后续任务先排队，队满后才尝试扩到 8；再无法接收就触发拒绝。实际还需监控排队时长与下游容量。

**追问：为什么调大最大线程数可能没效果？** 采用无界队列时，核心线程达到后任务通常直接排队，队列不满就不会因为排队继续扩到 maximumPoolSize。应结合队列等待、下游承载和拒绝指标设计容量。

### 9 四种拒绝策略分别怎样处理任务

<span class="java-label java-insight">理解与追问：</span> 支付等不能静默丢弃的任务，应让提交方感知拒绝并选择限流、落库或可靠重试。CallerRunsPolicy 虽然能够向提交端施加压力，但调用线程可能是重要的网络事件线程，应先评估阻塞影响。

`ThreadPoolExecutor` 关闭后，或容量饱和而无法接收任务时，会触发拒绝处理。四种内置策略分别是：

| 拒绝策略 | 如何处理任务 |
| --- | --- |
| `AbortPolicy` | 抛出 `RejectedExecutionException`，让提交方感知拒绝。 |
| `DiscardPolicy` | 直接丢弃任务。 |
| `DiscardOldestPolicy` | 线程池未关闭时，移除队头任务，再尝试提交当前任务。 |
| `CallerRunsPolicy` | 线程池未关闭时，由提交任务的调用者线程直接执行。 |

- CallerRuns 的调用者可能是请求线程、消息消费线程或另一个工作线程,并不固定是 main 线程;线程池关闭后它也会丢弃任务。

- DiscardOldest 丢的是队头,只有 FIFO 队列中才可简单理解为最早入队;重试仍可能失败。

- 任务不能丢时,需要可观测的拒绝反馈、限流或可靠补偿;静默丢弃 FutureTask 可能让 get 长期等待。

<span class="java-label java-warning">易错提醒：</span> 拒绝不只在“队列满”时发生,关闭状态也会拒绝新任务。

<span class="java-label java-example">具体场景：</span> 如果提交者是 HTTP 工作线程，CallerRunsPolicy 会让它自己执行任务，响应时间随之变长；若提交者是事件循环，还可能影响其他连接。关键任务可选择明确拒绝、上游限流和可恢复补偿。

**追问：CallerRunsPolicy 一定安全吗？** 不一定。它会让提交者承担执行时间，在请求线程上可能拉长响应，在事件循环上可能阻塞其他连接；线程池已关闭时也不会继续执行被拒任务。

### 10 线程池有哪些状态 怎样正确关闭

<span class="java-label java-insight">理解与追问：</span> 服务关闭时可先停止接收新流量，调用 shutdown，再等待任务完成；超时后是否 shutdownNow，要结合任务可中断性与补偿方案。还应确认未开始的排队任务如何处理。

线程池有五种状态：

- **`RUNNING`**：接收并处理任务。
- **`SHUTDOWN`**：不接收新任务，但继续处理已入队任务。
- **`STOP`**：不再处理队列任务，并尝试中断工作线程。
- **`TIDYING`**：所有任务结束、工作线程数为零后进入该状态，执行终止钩子。
- **`TERMINATED`**：终止钩子执行完毕后的最终状态。

- shutdown 发起有序关闭;shutdownNow 尝试中断执行中的任务并返回尚未开始的排队任务。

- 关闭方法返回不代表所有任务已经完成,需要 awaitTermination 等方式等待并处理超时。

- 任务不响应中断时,shutdownNow 也不能保证线程池立刻终止;状态转换并不要求依次经过全部五个状态。

<span class="java-label java-warning">易错提醒：</span> 最终状态名称是 TERMINATED,不是 TERMINAL。

<span class="java-label java-example">具体场景：</span> 服务停机时先停止接收新请求，再 shutdown 并等待一段时间；仍未完成时按业务决定是否尝试 shutdownNow。对返回的未开始任务要安排恢复，对正在运行且不响应中断的任务不能假定已结束。

**追问：shutdownNow 能否立即结束所有任务？** 不能。它尝试中断正在运行的任务并返回未开始的队列任务；任务若不响应中断可能继续运行。停机流程还要限定等待时间并安排补偿。

### 11 AQS 是什么 常见工具怎样使用 state

<span class="java-label java-insight">理解与追问：</span> 面试中可沿 state、排队、阻塞和唤醒四步解释，而不是把 AQS 等同于一把锁。以信号量为例，state 对应许可数量；获取失败才可能排队等待。

AQS 是构建锁和同步器的框架,通过原子更新的 state、等待队列及阻塞唤醒机制组织竞争,支持独占和共享模式。state 的业务含义由子类定义,不能把它统一理解成“是否有锁”。队列按先进先出组织,也不意味着所有基于 AQS 的工具自动公平。

- ReentrantLock 用 state 表示重入次数,还要记录拥有者;同一线程可以重入,不是 state 大于零就必须阻塞。

- CountDownLatch 用剩余计数控制等待,减到零后释放等待者;Semaphore 用许可数量控制申请,许可不足才需等待。

- JDK 8/21 的 ReentrantReadWriteLock 将 int state 高 16 位表示读计数,低 16 位表示写重入计数;这是特定版本实现细节。

- JDK 8 的 FutureTask 已不基于 AQS,而是用 CAS 更新状态并维护等待线程栈,不能把旧版实现套到新版本。

<span class="java-label java-warning">易错提醒：</span> AQS 字段名是 state,且“释放后唤醒”不等于被唤醒线程必然立即取得资源。

<span class="java-label java-example">具体场景：</span> CountDownLatch 初始计数为 3，主线程 await；三个工作各完成一次 countDown，计数降到 0 后等待者才可继续。它的 state 表示剩余次数，而 ReentrantLock 的 state 在其实现中表示重入计数。

**追问：AQS 的 state 是不是锁标志？** 它只是同步状态，具体含义由实现决定。重入锁用它表示持有次数，CountDownLatch 用它表示剩余计数；队列按顺序组织也不代表所有实现都公平。

### 12 Java SPI 怎样发现实现

<span class="java-label java-insight">理解与追问：</span> 例如支付系统定义 PaymentProvider 接口，不同厂商把实现声明在服务配置中，应用通过 ServiceLoader 发现。需要多实现并存时，还得另外设计名称匹配、优先级和加载失败处理。

SPI 是服务提供者接口机制:使用方依赖服务接口,提供方按约定声明实现,ServiceLoader 在运行时查找并按需加载提供者,从而降低使用方对具体实现的直接依赖。它提供发现能力,具体选择哪个实现、如何排序仍由使用方或框架决定。

- 类路径方式在 META-INF/services/接口全限定名 文件中逐行写实现类全限定名;对应实现通常需要公开无参构造。

- JDK 9 模块化方式可通过 uses、provides ... with ... 声明服务与提供者。

- 发现过程受类加载器可见性影响;ServiceLoader 实例本身不保证多线程并发使用安全。

<span class="java-label java-warning">易错提醒：</span> Dubbo 等框架的 SPI 扩展规则不能直接当成 JDK ServiceLoader 的规则。

<span class="java-label java-example">具体场景：</span> 定义 PaymentProvider 接口，A、B 厂商分别提供实现，并在 `META-INF/services/接口全限定名` 声明。ServiceLoader 可以发现实现；应用还要决定按渠道如何选择、重复实现如何处理。

**追问：SPI 发现了多个实现，默认选谁？** ServiceLoader 提供发现与加载机制，业务仍需自己定义筛选、冲突和排序规则。还要确认配置路径、类加载器可见性和实现初始化失败时如何处理。

### 13 反射和 JavaBeans 内省是什么关系

<span class="java-label java-insight">理解与追问：</span> 反射回答“有哪些方法、如何调用”；JavaBeans 内省进一步识别逻辑属性。比如 getName 和 setName 可以被识别为 name 属性，但字段名与属性访问器不必完全按反射字段直接映射。

反射通过 Class 对象及 Method、Field、Constructor 等 API,在运行时检查类型信息并调用成员。Class 是 Java 层的类型表示,不等同于 HotSpot 内部的 InstanceKlass。JavaBeans 的Introspector 在反射等机制上识别 Bean 属性、读写方法和事件,是更高层的内省工具。

- getBeanInfo 返回 BeanInfo,可从 PropertyDescriptor 取得属性名及 getter、setter,适合实现属性映射等功能。

- 访问非公开成员仍受访问控制与模块封装限制;setAccessible 不是在所有 JDK、所有模块上都能成功。

- JDK 9 已移除 rt.jar;JDK 18 核心反射改为基于方法句柄实现,不能把早期内部机制当作长期规范。

<span class="java-label java-warning">易错提醒：</span> 反射是否更慢取决于版本、调用方式和优化条件,不能直接背固定性能倍数。

<span class="java-label java-example">具体场景：</span> 例如 `getName()` 和 `setName(String)` 可以按 JavaBeans 约定构成 name 属性，映射工具可通过 PropertyDescriptor 访问；直接反射 Field 则是在检查字段，不一定与属性一一对应。

**追问：Bean 属性一定对应同名字段吗？** 不一定。JavaBeans 内省主要依据访问方法等约定识别属性；反射读取字段则是另一层操作。序列化或映射框架采用哪种方式，需要查看其配置。

<span id="14-新增--equals-hashcode-的契约及-string-不可变性"></span>

### 14 == equals hashCode 的契约及 String 不可变性

<span class="java-label java-insight">理解与追问：</span> 例如两个内容相同但分别创建的字符串，equals 可以为 true，而引用比较未必相同。自定义对象作为 HashMap 键时，加入之后再修改参与相等判断的字段，可能导致后续查找失败。

基本类型的 == 比较值,引用类型的 == 比较是否指向同一对象;equals 定义逻辑相等,Object 默认仍比较对象身份。重写 equals 通常必须同时重写 hashCode,使逻辑相等对象具有相同哈希值。String 按内容比较,创建后的字符串内容不能通过正常 API 修改。

- equals 应满足自反、对称、传递和一致性,非空对象与 null 比较应返回 false;参与相等判断的信息未变时,hashCode 应保持一致。哈希相同仍可能是不同对象,不能用哈希值代替相等判断。

- String 不可变依赖禁止被继承、封装内部存储以及不暴露可修改内容等设计,便于共享和用作键。拼接、替换等操作返回结果字符串,原对象不变;变量重新赋值只是改了引用。

<span class="java-label java-warning">易错提醒：</span> final 引用不能重新赋值,不代表它引用的任意对象都不可变;字符串内容不要用 == 判断。

<span class="java-label java-example">具体场景：</span> 两个新建的 String 内容都是 `abc`：equals 按内容可为 true，引用 `==` 不一定为 true。自定义 UserKey 如果 equals 依赖 id，hashCode 也应基于相同的相等信息，且作为 Map 键时不要修改该信息。

**追问：equals 相同但 hashCode 不同会怎样？** 违反约定后，哈希容器可能在不同桶查找，导致逻辑上相等的键无法按预期命中。作为键后也不要改变参与相等与哈希计算的字段。

<span id="15-新增-concurrenthashmap-如何保证线程安全"></span>

### 15 ConcurrentHashMap 如何保证线程安全

<span class="java-label java-insight">理解与追问：</span> 可以把桶理解为缩小更新竞争的粒度：不同桶的写入通常不需要同一把全局锁。但跨两个键的转账不因此变成原子操作，仍需额外事务或同步设计。

JDK 8 的 ConcurrentHashMap 以数组、链表和树组织数据,空桶插入常用 CAS,非空桶更新结合桶级同步,并用必要的可见性机制支持并发读取;扩容时多个线程可协作迁移。它主要提供单键操作的并发保证,不能把一连串业务操作自动变成事务。

- 不要把 JDK 7 的 Segment 分段锁模型直接套到 JDK 8;get 通常不需要获取更新时的桶监视器。键和值均不能为 null,成功读取某键更新后的非空值时,能看到该次更新之前的相关写入。

- 先检查再 put 应按语义改成 putIfAbsent、compute 或 merge 等原子操作;回调宜短小且避免递归修改同一映射。迭代是弱一致的,size 等聚合结果在并发修改期间不能充当精确控制依据。

<span class="java-label java-warning">易错提醒：</span> 容器线程安全不等于存入其中的可变对象自动线程安全,也不保证跨多个键的一致性。

<span class="java-label java-example">具体场景：</span> 缓存容器安全不代表缓存值安全：多个线程从 ConcurrentHashMap 拿到同一个普通 ArrayList 后同时 add，列表仍可能出错。可根据需要使用并发集合、锁或不可变值整体替换。

**追问：putIfAbsent 后修改 value 是否安全？** 容器保护的是映射操作，不自动保护存进去的可变对象。若多个线程修改同一个普通列表，仍要使用安全的数据结构或对该对象另行同步。

<span id="16-新增-threadlocal-原理及线程池中的泄漏风险"></span>

### 16 ThreadLocal 原理及线程池中的泄漏风险

<span class="java-label java-insight">理解与追问：</span> 典型场景是在线程池中保存请求用户信息：线程执行完请求后还会复用，因此必须在 finally 中清除。若只依赖弱引用回收，仍可能残留值或使下一次任务读到旧上下文。

ThreadLocal 为当前线程关联一份变量值,JDK 8 中数据实际保存在 Thread 持有的ThreadLocalMap 内,键是 ThreadLocal 的弱引用,值是强引用。线程池工作线程长期存活,任务结束后若未清理,值可能持续被保留,也可能被后续任务错误读到,形成上下文串用。

- 键失去其他强引用后可能被回收,但值不会因此立即回收;后续部分 get、set、remove 路径会清理失效项,不能依赖不确定的清理时机。即使 ThreadLocal 是静态字段,未 remove 的大对象也可能长期占用内存。

- 在任务边界用 try/finally 执行 remove,嵌套上下文则按约定恢复旧值。普通 ThreadLocal 不自动跨线程传递;InheritableThreadLocal 通常在线程创建时继承,不能可靠代表线程池每次提交任务时的上下文。

- **新版补充：** 虚拟线程同样支持 ThreadLocal，但其数量可能很大，不宜默认使用 ThreadLocal 为每个虚拟线程缓存昂贵资源。JDK 24 对 `synchronized` 引起的虚拟线程固定问题进行了改进，不能把旧版本限制直接套用到新版；应按实际 JDK 和阻塞路径排查。参考 [Oracle 虚拟线程指南](https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html)。

<span class="java-label java-warning">易错提醒：</span> ThreadLocal 只隔离每线程的变量关联;多个线程若存入同一个可变对象,仍会共享该对象。

<span class="java-label java-example">具体场景：</span> 线程池线程先处理用户 A，再处理用户 B；A 的用户上下文若未清除，B 可能读取到旧值。请求处理结束应在 finally 中 remove；有嵌套上下文时需按约定恢复此前的值。

**追问：弱引用是否意味着不会泄漏？** 不是。ThreadLocalMap 的键是弱引用，值是强引用；线程长期存活且没有触发清理时，值可能被保留。在线程池任务边界用 finally remove 更可靠。

<span id="17-新增-cas-原理-aba-问题与适用边界"></span>

### 17 CAS 原理 ABA 问题与适用边界

<span class="java-label java-insight">理解与追问：</span> 例如库存数量从 5 到 4 再补回 5，只比较数值时看不出中间变化。若业务只关心当前库存，ABA 未必造成问题；若必须识别期间发生过修改，则需要版本或其他状态约束。

CAS 将内存位置的当前值与预期值比较,相等才原子地写入新值,否则返回失败。调用方可重新读取并计算后重试,形成乐观更新。ABA 指值从 A 变成 B 又变回 A,比较仍成功,却遗漏了中间变化;只有业务依赖变化历史时才构成正确性问题。

- AtomicStampedReference 可把引用和版本戳一起比较更新,前提是修改者遵循版本递增约定;读取引用和版本也应保持配对。版本戳不是自动防错,错误复用或回绕仍需按业务评估。

- CAS 重试在竞争激烈时可能消耗较多 CPU,不能保证每个线程公平或限时成功;重试计算应避免不可重复的外部副作用。多个字段的不变量可封装为不可变状态整体替换,或直接使用适当的锁。

<span class="java-label java-warning">易错提醒：</span> 一次 CAS 只提供相应位置的原子更新,不自动保证整个方法、多个独立变量或数据库操作原子。

<span class="java-label java-example">具体场景：</span> 假设库存值从 5 改为 4，又被补回 5。只比较当前值的 CAS 可能成功，但看不到中间变更；只有业务依赖这段变更历史时才需要版本戳等额外机制，版本也需按协议更新。

**追问：ABA 一定会导致错误吗？** 不一定。若业务只关心当前位置的当前值，中间变化可能无关紧要；若依赖变化历史，就需要版本戳等机制，并约束所有写入者共同遵守。

<span id="18-新增-死锁的必要条件-预防与定位"></span>

### 18 死锁的必要条件 预防与定位

<span class="java-label java-insight">理解与追问：</span> 例如线程 A 先持有订单锁再等库存锁，线程 B 反向持有，可能形成环。通过统一锁顺序降低风险；定位时需查看多个线程的持锁与等待关系，而不是仅凭一个 WAITING 状态判断。

典型资源死锁需要同时具备互斥、占有且等待、不可剥夺和循环等待四个条件。例如两个线程分别持有两把锁,并等待对方释放。预防可破坏其中一个条件,工程中常用统一加锁顺序、缩小持锁范围,以及获取失败时释放已持有资源后重试。

- 涉及多资源时可按稳定唯一标识统一排序后加锁;使用 tryLock 超时要配合释放、回滚与退避,避免双方不断互让形成活锁。不要持有业务锁等待不受控的远程调用,也不能把超时本身当作正确释放资源的证明。

- 通过多次线程转储查看持锁者与等待链;ThreadMXBean.findDeadlockedThreads 可检查监视器及可拥有同步器的锁等待环。线程池任务互相等待、外部系统循环依赖等逻辑死锁,不一定能被该接口自动发现。

<span class="java-label java-warning">易错提醒：</span> BLOCKED 或 WAITING 状态本身不证明死锁;interrupt 也不能强制解除所有锁等待。

<span class="java-label java-example">具体场景：</span> 线程 A 持有订单锁等待库存锁，线程 B 持有库存锁等待订单锁，两条等待边形成环。让所有路径按同一顺序获取锁可降低此类死锁；定位时要查看完整持锁和等待关系。

**追问：等待很久就是死锁吗？** 不是。死锁需要等待关系形成环；慢调用或长事务也会造成等待。采集多次线程转储并还原持锁者与等待链，才能判断是否存在环。

## 参考资料

- [Oracle ArrayList API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ArrayList.html)
- [Oracle ConcurrentHashMap API](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html)
- [Oracle ThreadPoolExecutor API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)

- [Java 8 ThreadPoolExecutor 文档](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html)
- [MySQL 8.0 InnoDB 隔离级别](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)与 [锁机制](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
- [Redis 内存淘汰策略](https://redis.io/docs/latest/develop/reference/eviction/)
- [Spring 事务注解与代理](https://docs.spring.io/spring/reference/6.2/data-access/transaction/declarative/annotations.html)

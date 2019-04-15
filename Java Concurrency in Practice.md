# Java Concurrency in Practice

线程的优势：

+ **发挥多处理器的强大能力**。单线程程序同时只能在一个处理器上运行，其他资源将被浪费。多线程程序可以同时在多个处理器上执行。提升系统吞吐率。
+ **建模的简单性**。只包含一种类型的任务比包含多种不同类型任务的程序要更易于编写，错误更少，也更容易测试。如果为模型中每种类型的任务都分配一个专门的线程，那么可以形成一种串行执行的假象。
+ **异步时间的简化处理**。
+ **响应更灵敏的用户界面**。

线程的风险：

+ **安全性问题**。在没有充足同步的情况下，多个线程中的操作执行顺序时不可预测的。
+ **活跃性问题**。某个操作无法继续执行下去（死锁、饥饿、活锁）。
+ **性能问题**。挂起活跃线程并转而运行另一个线程会频繁地出现上下文切换操作。

## 线程安全性

要编写线程安全的代码，其核心在于要对状态（特别是对共享的和可变的）访问操作进行管理。

`HashMap`的状态不仅存储在`HashMap`对象本身，还存储在许多`Map.Entry`对象中。对象的状态包含可影响其外部可见行为的任何数据。

要使对象成为线程安全的，需要同步机制来协调对其可变状态的访问。“同步”术语包括`synchronized`，`volatile`，显示锁以及原子变量。

### 安全性

线程安全性的定义中，最核心的概念就是正确性。正确性的含义是，某个类的行为与其规范完全一致，“所见即所知”。

**定义**：当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

在**线程安全类**的对象实例上执行的任何串行或并行操作都不会使对象处于无效状态。线程安全类中封装了必要的同步机制，因此客户端无须进一步采取同步措施。

无状态对象一定是线程安全的。

### 原子性

一组语句作为一个不可分割的单元被执行。

#### 竞态条件

当某个计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件。（正确的结果取决于运气）。最常见的竞态条件类型就是“先检查后执行（Check-Then-Act）”操作，通过一个可能失效的观测结果来决定下一步动作。

+ 延迟初始化

    ```java
    public class LazyInitRace {
        private ExpensiveObject instance = null;
        public ExpensiveObject getInstance() {
            // 多个线程有可能同时进入if并生成多个对象
            if (instance == null) instance = new ExpensiveObject();
            return instance;
        }
    }
    ```

+ 读取-修改-写入 `i++`

#### 复合操作

要避免竞态条件问题，就必须在某个线程修改该变量时，通过某种方式防止其他线程使用这个变量，从而确保其他线程只能在修改操作完成之前或之后读取和修改状态，而不是在修改的过程中。

“先检查后执行”以及“读取-修改-写入”等操作统称为复合操作：包含了一组必须以原子方式执行的操作以确保线程安全性。

递增操作`++count`是一个复合操作。这是一个“读取-修改-写入”的操作序列，并且其结果状态依赖于之前的状态。

当在无状态类中添加一个状态时，如果该状态完全由线程安全的对象来管理，那么这个类仍然是线程安全的。当状态变量的数量由一个变为多个时，并不会像由零个变为一个那样简单。

在可行的情况下，使用现有的线程安全对象（如`AtomicLong`）来管理类的状态。与非线程安全对象相比，线程安全对象的可能状态和状态转换更简单，这使得维护和验证线程安全更容易。

### 加锁机制

线程安全性的定义中要求，多个线程之间的操作无论采用何种执行时序或交替方式，都要保证不变性条件不被破坏。当不变性条件中涉及多个变量时，它们不是独立的：某个变量的值会约束其他变量的允许值。因此，当更新某个变量时，必须在同一原子操作中更新其他的变量。

#### 内置锁

以关键字`synchronized`来修饰的方法就是一种横跨整个方法体的同步代码块，其中的锁就是方法调用所在的对象。静态的`synchronized`方法以`Class`对象作为锁。

每个Java对象都可以用做一个实现同步的锁，被称为内置锁或监视器锁。是一种互斥锁。

内置锁是可重入的。线程试图获取它已经持有的锁，会请求成功。“重入”意味着获取锁的操作的粒度是“线程”，而不是“调用”。

### 用锁来保护状态

锁能够使其保护的代码路径以串行形式访问，因此可以通过锁来构造协议以保证对共享状态的独占访问。遵循这些协议可始终确保状态的一致性。

在复合操作的整个执行过程中持有锁可以使复合操作成为原子操作。但是，仅使用`synchronized`块包装复合操作是不够的，如果用同步来协调对变量的访问，则在访问变量的任何地方都需要同步，且必须使用相同的锁。

常见的锁定约定是在对象中封装所有可变状态，并通过使用对象的内部锁同步访问可变状态的任何代码路径来保护它免受并发访问。许多线程安全类使用此模式，例如`Vector`和其他同步集合类。通过添加新方法或代码路径并忘记使用同步，也很容易意外破坏此锁定协议。

当类的不变性条件涉及多个状态变量时，还有另外一个需求：在不变性条件中的每个变量都必须由同一个锁来保护。

如果只是将每个方法都作为同步方法，并不足以确保复合操作都是原子的：

```java
if (!vector.contains(element))
    vector.add(element);
```

虽然同步方法可以使单个操作成为原子，但是当多个操作组合成复合操作时，需要额外的锁定。

### 活跃性和性能

要判断同步代码块的合理大小，需要在各种设计需求之间进行权衡，包括安全性（必须得到满足）、简单性和性能。

## 对象的共享

加锁的含义不仅仅局限于互斥行为，还包括内存可见性。

### 可见性

在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行调整，导致可见性问题。

内部锁可用于保证一个线程以可预测的方式看到另一个线程的影响。

字段被声明为`volatile`时，编译器和运行时会注意到该变量是共享的，并且对它的操作不应该与其他内存操作重新排序。`volatile`变量不会缓存在寄存器或缓存中，因此读取`volatile`变量始终会返回任何线程的最新写入。

```java
// 典型用法
volatile boolean asleep;
while (!asleep)
    countSomeSheep();
```

仅当满足以下所有条件时，才能使用`volatile`变量：

+ 写入变量不依赖于其当前值，或者能确保只有一个线程更新该值。
+ 变量不参与其他状态变量的不变性条件。
+ 访问变量时不需要加锁。

### 发布与逸出

**发布（Publish）**：使对象能够在当前作用域之外的代码中使用。
**逸出（Escape）**：某个不应该发布的对象被发布。

当某个对象逸出后，你必须假设有某个类或线程可能会误用该对象。这正是需要使用封装的最主要原因：封装能够使得对程序的正确性进行分析变得可能，并使得无意中破坏设计约束条件变得更难。

不要在构造过程中使`this`引用逸出。当对象在其构造函数创建一个线程时，`this`会被新创建的线程共享。在对象尚未完全构造之前，新的线程就可以看见它。

### 线程封闭

仅在单线程内访问数据，就不需要同步。这种技术被成为线程封闭。当某个对象封闭在一个线程中时，这种用法将自动实现线程安全性，即使被封闭的对象本身不是线程安全的。

+ Ad-hoc线程封闭：维护线程封闭性的职责完全由程序实现来承担。（脆弱的）
+ 栈封闭：对象只能通过局部变量访问。要确保对象不会逸出。
+ ThreadLocal类

### 不变性

对象在创建后状态就不能修改，就称为不可变对象。线程安全性是不可变对象的固有属性之一。

在Java语言规范和Java内存模型中都没有给出不可变性的正式定义，但不可变性并不等于将对象中所有的域都声明为`final`类型，即使对象中所有的域都是`final`类型的，这个对象也仍然是可变的，因为在`final`类型的域中可以保存对可变对象的引用。

当满足以下条件时，对象才是不可变的：

+ 对象创建以后其状态就不能修改。
+ 对象的所有域都是`final`类型。
+ 对象是正确创建的（在对象的创建期间，this引用没有逸出）。

在不可变对象的内部仍可以使用可变对象来管理它们的状态。（这里主要注意第一个条件）

```java
public final class ThreeStooges {
    private final Set<String> stooges = new HashSet<String>();
    public ThreeStooges() {
        stooges.add("Moe");
    }
    public boolean isStooge(String name) {
        return stooges.contains(name);
    }
}
```

#### Final域

`final`类型的域是不能修改的（但如果引用的对象是可变的，那么这些对象的状态可以修改）。在Java内存模型中，`final`域还有着特殊的意义，能确保初始化过程的安全性，从而可以不受限制地访问不可变对象，并在共享这些对象时无须同步。

即使一个对象是可变的，通过将对象的某些域声明为`final`类型，仍然可以简化对其状态的推理，因为限制对象的可变性，相当于限制了它的可能状态集。

正如**除非需要更高的可见性，否则应将所有的域都声明为私有域**是一个良好的编程习惯，**除非需要某个域是可变的，否则应将其声明为`final`域**也是一个良好的编程习惯。

### 安全发布

#### 不可变对象与初始化安全性

Java内存模型为不可变对象的共享提供了一种特殊的初始化安全性保证。

> 对于含有`final`域的对象，JVM必须保证对对象的初始引用在构造函数之后执行，不能乱序执行（out of order），也就是可以保证一旦你得到了引用，`final`域的值都是完成了初始化的，也就是书中所说的“初始化安全性”的保证。[来源](https://blog.csdn.net/ll530304349/article/details/52629254)

#### 安全发布的常用模式

要安全地发布对象，必须同时使对象的引用和状态对其他线程可见。正确构造的对象可以通过以下方式安全发布：

+ 从静态初始化程序初始化对象引用;
+ 将对它的引用存储到`volatile`类型的域或`AtomicReference`中;
+ 将对它的引用存储到正确构造的对象的`final`类型域中;
+ 将对它的引用存储到由锁保护的域中。

#### 事实不可变对象

在技​​术上来看是可变的，但在发布后不会对其状态进行修改的对象被称为“事实不可变对象”。使用事实不可变对象可以通过减少同步需求来简化开发并提高性能。

例如，`Date`是可变的，但是如果你使用它就好像它是不可变的那样，你可以消除在跨线程共享`Date`时需要的锁。

```java
public Map<String, Date> lastLogin = Collections.synchronizedMap(new HashMap<String, Date>());
```

#### 可变对象

如果在对象构造之后可以修改，则安全发布仅确保“发布当时”状态的可见性。不仅发布可变对象需要使用同步，而且还必须每次访问对象时使用，以确保后续修改的可见性。要安全地共享可变对象，必须安全地发布它们，并且要么是线程安全的，要么是锁保护起来。

对象的发布要求取决于其可变性：

+ 可通过任何机制发布不可变对象;
+ 必须安全地发布事实不可变的对象;
+ 必须安全发布可变对象，并且必须是线程安全的或由锁保护。

#### 安全地共享对象

当获得对象的引用时，应该知道可以使用它做什么。在使用之前需要锁吗？允许修改它的状态，还是只读？许多并发错误源于未能理解共享对象的这些“参与规则”。发布对象时，应记录如何访问对象。

在并发程序中使用和共享对象的最有用的策略是：

+ **线程封闭**。对象由一个线程专有，并且只可以通过其拥有的线程进行修改。
+ **只读共享**。对象可以由多个线程同时访问，无需额外同步，但不能由任何线程修改。共享只读对象包括不可变对象和事实不可变对象。
+ **线程安全共享**。对象在内部实现同步，因此多个线程可以通过其公共接口自由访问它，而无需进一步同步。
+ **保护对象**。对象只有在持有特定锁的情况下才能访问。包括封装在其他线程安全对象中的对象和已知由特定锁保护的已发布对象中的对象。

## 对象的组合

### 设计线程安全的类

线程安全类的设计过程应包括以下三个基本元素：

+ 找出构成对象状态的变量;
+ 找出约束状态变量的不变性条件;
+ 建立管理对象状态的并发访问的策略。

如果对象中所有的域都是基本类型的变量，那么这些域将构成对象的全部状态。如果在对象的域中引用了其他对象，那么该对象的状态将包含被引用对象的域。例如，`LinkedList`的状态就包括该链表中所有节点对象的状态。

同步策略定义对象如何协调对其状态的访问而不违反其不变性条件或后置条件。它规定了不变性，线程封闭和加锁机制的组合用于维护线程安全性，以及哪些变量由哪些锁保护。要确保可以分析和维护类，就要记录同步策略为文档。

## 基础构建模块

### 同步容器类

同步容器类包括`Vector`和`Hashtable`，以及由`Collections.synchronizedXxx`工厂方法创建的同步包装器类。 这些类通过封装其状态，并对每个公共方法进行同步来实现线程安全，这样一次只有一个线程可以访问容器的状态。

同步容器类的复合操作存在竞态条件：迭代、跳转（根据指定顺序找到当前元素的下一个元素）以及条件运算（若没有则添加）。要让这些复合操作成为原子操作。

```java
public static Object getLast(Vector list) {
    synchronized(list) {
        int lastIndex = list.size() - 1;
        return list.get(lastIndex);
    }
}

public static void deleteLast(Vector list) {
    synchronized(list) {
        int lastIndex = list.size() - 1;
        list.remove(lastIndex);
    }
}

synchronized(vector) {
    for (int i = 0; i < vector.size(); i++)
        doSomething(vector.get(i));
}
```

标准容器的`toString`方法间接的迭代容器。

`hashCode`和`equals`方法也会间接执行迭代，如果容器用作另一个容器的元素或键，则可以调用它。 类似地，`containsAll`，`removeAll`和`retainAll`方法，以及将容器作为参数的构造函数，也会迭代。 所有这些迭代的间接使用都可能导致`ConcurrentModificationException`。

### 并发容器

同步容器通过串行化对容器状态的所有访问来实现其线程安全性。这种方法的成本是不良的并发性。当多个线程争用容器的锁时，吞吐量会受到影响。

+ `ConcurrentHashMap`替代同步且基于散列的`Map`, 新的`ConcurrentMap`接口增加了对常见复合操作的支持

    使用分段锁来提供更好的并发性和可伸缩性。任意多个读取线程可以并发访问，一定数量的写入线程可以并发修改。迭代器具有弱一致性，`size`、 `isEmpty`的语义被减弱，只是估计值。支持常见复合操作：

    ```java
    // 仅当k没有对应的映射才插入
    v putIfAbsent(K key, V value);
    // 仅当k被映射到v时才移除
    boolean remove(K key, V value);
    // 仅当k被映射到oldValue时才替换为newValue
    boolean replace(K key, V oldValue, V newValue);
    // 仅当k被映射到某个值时才替换为newValue
    V replace(K key, V newValue);
    ```

+ `CopyOnWriteArrayList`替代同步`List`，`CopyOnWriteArraySet`替代同步`Set`

    每次修改时创建并重新发布一个新的容器副本从而实现可变性。多个线程可以同时迭代而不会干扰。因为创建副本存在一定开销，仅当迭代操作远远多于修改时才使用。

+ `Queue`上的操作不会阻塞，如果队列为空，那么获取操作将返回空值
+ `BlockingQueue`扩展了`Queue`，如果队列为空，会阻塞直到队列出现可用元素

    非常适合于“生产者-消费者”模式。`Executor`基于此模式。`LinkedBlockingQueue`和`ArrayBlockingQueue`是FIFO队列，分别类似`LinkedList`和`ArrayList`。`PriorityBlockingQueue`按优先级排序的队列。`SynchronousQueue`维护一组线程，这些线程等待把元素加入或移出队列。

    包含足够的内部同步机制从而安全地将对象从生产者线程发布到消费者线程。

+ `ConcurrentSkipListMap`和`ConcurrentSkipListSet`分别替代`SortedMap`和`SortedSet`
+ `Deque`和`BlockingDeque`分别扩展`Queue`和`BlockingQueue`。双端队列实现了在头和尾的高效插入和移除。实现包括`ArrayDeque`和`LinkedBlockingDeque`。

    适用于工作密取（Work Stealing)。每个消费者拥有各自的双端队列。如果消费者完成自己双端队列中的全部工作，可以从其他消费者双端队列末尾秘密地获取工作。具有更高的可伸缩性，不会在单个共享队列上发生竞争，极大地减少了竞争。

### 同步工具类

+ `CountDownLatch`。闭锁到达结束状态之前，其他线程会阻塞，直到闭锁到达结束状态。
+ `Semaphore`。信号量用来控制同时访问某个资源的操作数量。
+ `CyclicBarrier`。所有线程必须同时到达栅栏位置才能继续执行。

## 任务执行

### 在线程中执行任务

串行处理机制通常无法提供高吞吐率或快速响应性。

正常负载情况下，“为每个任务分配一个线程”能提升串行执行的性能。存在缺陷：

+ 线程生命周期的开销非常高。创建和销毁需要消耗大量资源。
+ 资源消耗。线程数量多于可用处理器数量，那么有些线程将闲置。大量空闲线程会占用内存，且存在大量竞争。
+ 稳定性。大量创建线程可能抛出`OutOfMemoryError`。

### Executor框架

```java
public interface Executor {
    void execute(Runnable command);
}
```

通过将任务的提交与执行解耦开来，从而无须太大的困难就可以为某种类型的任务指定和修改执行策略。

通过重用现有的线程而不是创建新线程，可以在处理多个请求时分摊在线程创建和销毁中产生的巨大开销。另一个额外的好处是，当请求到达时，线程通常已经存在，因此不会由于等待创建而延迟任务的执行，从而提高响应性。

+ `newFixedThreadPool`。固定长度的线程池。每提交一个任务时就创建一个线程，直到达到最大数量。
+ `newCachedThreadPool`。可缓存的线程池。规模超过处理需求将回收空闲线程，而当需求增加时则添加新线程。规模不存在限制。
+ `newSingleThreadExecutor`。单线程的`Executor`，能确保依照任务在队列中的顺序来串行执行。
+ `newScheduledThreadPool`。固定长度的线程池，而且以延迟或定时的方式执行。

`Executor`扩展了`ExecutorService`接口，添加用于生命周期管理的方法：

```java
public interface ExecutorService extends Executor {
    // 平缓的关闭线程：不再接受任务，同时等待已经提交的执行完。包括未开始的
    void shutdown();
    // 粗暴的关闭：尝试取消所有运行的任务，不再启动尚未开始的
    List<Runnable> shutdownNow();
    boolean isShutdown();
    // 轮询是否已经终止
    boolean isTerminated();
    // 等待到达终止状态
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
}
```

## 中断和关闭

阻塞库方法(`Thread.sleep` `Object.wait`)都会检查线程何时中断，并且在发现中断时提前返回。它们在响应中断时执行的操作包括：清除中断状态，抛出`InterruptedException`，表示阻塞操作由于中断而提前结束。

## 线程池的使用

### 在任务与执行策略之间的隐性耦合

在单线程的`Executor`中，如果一个任务将另一个任务提交到同一个`Executor`，并且等待被提交任务的结果，那么通常会引发死锁。

当任务数超过了线程池的最大任务数，新的任务需要等待其他任务释放。

### 设置线程池的大小

对于**计算密集型**（消化CPU资源）的任务，在拥有N个处理器的系统上，当线程池的大小为N+1时，通常能实现最优的利用率。（即使当计算密集型的线程偶尔由于页缺失故障或者其他原因而暂停时，这个“额外”的线程也能确保CPU的时钟周期不会被浪费。）。对于包含**I/O操作**或者其他**阻塞操作**的任务，由于线程并不会一直执行，因此线程池的规模应该更大。

### 配置ThreadPoolExecutor

```java
public ThreadPoolExecutor(int corePoolSize, // 基本大小
                          int maximumPoolSize, // 最大大小
                          long keepAliveTime, // 线程的空闲时间超过存活时间，会被标记为可回收的
                          TimeUnit unit, // 存活时间单位
                          BlockingQueue<Runnable> workQueue, // 任务队列
                          ThreadFactory threadFactory, // 线程工厂（自定义线程名、Logger、统计信息）
                          RejectedExecutionHandler handler); // 饱和策略
```

`newFixedThreadPool`将线程池的基本大小和最大大小设置为参数中指定的值，而且创建的线程池不会超时。`newCachedThreadPool`将线程池打最大大小设置为`Integer.MAX_VALUE`，而将基本大小设置为零，并将超时设置为1分钟，可以无限扩展，并且当需求降低时会自动收缩。

`BlockingQueue`用来保存待执行的任务。三种基本方法：无界队列、有界队列和同步移交。有界队列有助于避免资源耗尽的情况发生，当队列填满后，需要**饱和策略**解决问题：

+ `AbortPolicy`。“中止”策略是默认的饱和策略。会抛出未检查的`RejectedExecutionException`。
+ `CallerRunsPolicy`。“调用者运行”策略。任务会在调用`execute`时在主线程中等待。
+ `DiscardPolicy`。“抛弃”策略。会抛弃新提交的任务。
+ `DiscardOldestPolicy`。“抛弃最旧的”策略。会抛弃下一个被执行的任务。

## 避免活跃性危险

### 死锁

当一个线程永远地持有一个锁，并且其他线程都尝试获得这个锁时，那么它们将永远被阻塞。

+ 锁顺序死锁

    两个线程试图以不同地顺序来获得相同地锁。

    ```java
    public class LeftRightDeedlock {
        private final Object left = new Object();
        private final Object right = new Object();

        public void leftRight() {
            sychronized(left) {
                sychronized(right) {
                    doSomething();
                }
            }
        }

        public void rightLeft() {
            synchronized(right) {
                synchronized(left) {
                    doSomethingElse();
                }
            }
        }
    }
    ```

+ 动态的锁顺序死锁

    ```java
    public void transferMoney(Account fromAccount, Account toAccount, DollarAmount amount) {
        synchronized(fromAccount) {
            synchronized(toAccount) {
                if (fromAccount.getBalance().compareTo(amount) < 0)
                    throw new InsufficientFundsException();
                else {
                    fromAccount.debit(amount);
                    toAccoutn.credit(amount);
                }
            }
        }
    }
    ```

+ 在协作对象之间发生的死锁

### 其他活跃性危险

+ 饥饿。当线程由于无法访问它所需要地资源而不能继续执行时。
+ 糟糕的响应性。CPU密集型地后台任务会与事件线程共同竞争CPU地时钟周期，对响应性造成影响。
+ 活锁。多个相互协作地线程都对彼此进行相应从而修改各自地状态，并使得任何一个线程都无法继续执行。解决活锁问题需要在重试机制引入随机性。

## 显式锁

### Lock与ReentrantLock

```java
public interface Lock {
    // 无条件锁，等于使用synchronize
    void lock();
    void lockInterruptibly() throws InterruptedException;
    // 可轮询，可定时锁
    boolean tryLock();
    boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

内置锁在功能上存在一些局限性（无法实现非阻塞结构）：

+ 无法中断一个正在等待获取锁的线程
+ 试图获得锁而不愿意永远等待它

`Lock`必须在`finally`块中释放锁。否则在被保护的代码中抛出异常，那么这个锁永远都无法释放。还必须考虑异常时，对象可能处于不一致状态（恢复不变性条件）。

`ReentrantLock`不能完全替代`synchronized`的原因：它更加危险，离开代码块不会自动清除锁。应该作为一种高级工具，在需要高级功能时才使用：可定时、可轮询、可中断，公平队列，非块结构。

#### 轮询锁与定时锁

内置锁出现死锁时恢复程序的唯一方法是重启，防止死锁的唯一方法是在构造程序时避免出现不一致的锁顺序。可定时、可轮询的锁提供了另一种选择：避免死锁发生。

轮询锁会在调用后立即返回，判断是否获得锁。可以使用轮询的方式来获取锁，并在循环了加上时间限制，来避免死锁发生。定时锁能如果不能在指定时间内给出结果，就会返回。

### 公平性

`ReentrantLock`的构造函数提供了两种公平性选择：创建非公平锁（默认）或公平锁。

+ 公平锁：线程按照它们发出请求的顺序来获得锁
+ 非公平锁：线程请求锁时，如果在发出请求的同时锁状态可以用，将跳过队列中的线程并获得锁（插队）

在非公平锁中，只有当锁被某个线程持有时，新发出请求的线程才会被放入队列中。

使用非公平锁的原因：可以提高性能。如果队列存在线程等待获得锁，接下来的请求也将会放到的队列中排队等待（挂起）。挂起-恢复这个过程存在着开销，而极大地降低性能。如果使用非公平锁，那么在发出请求时如果可用将直接获得，跳过了挂起-恢复的过程。在实际情况中，通常只需要保证被阻塞的线程能最终获得锁即可。依赖于公平的排队算法来确保正确性的情况并不常见。（不必要的话，不要为公平性付出代价）

恢复一个被挂起的线程与该线程真正开始运行之间存在着严重的延迟。

内置锁和默认的`ReentrantLock`一样不会提供确定的公平性保证。Java语言规范没有要求JVM以公平的方式来实现内置锁，而在各种JVM中也没有这样做。

### 读-写锁

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

`ReentrantLock`是一种互斥锁，互斥通常是一种过于强硬的加锁机制，保守的加锁策略。虽然可以避免“写/写”，“写/读”冲突，但同样也避免了“读/读”冲突。而有些数据结构大多数访问操作都是读操作，允许多个执行读线程同时访问，将会提升性能。只要每个线程都能确保取到最新的数据，并且在读取数据时不会有其他的线程修改数据，那么就不会发生问题。一个资源可以被多个读操作访问，或者被一个写操作访问，但两者不能同时进行。

> `volatile`也能在一写多读的情况提供跟`ReadWriteLock`一样的特性，但`ReadWriteLock`还可以在多个写线程的情况下使用，写线程互斥。

与`Lock`一样，`ReadWriteLock`可以采用多种实现方式，这些方式在性能、调度保证、优先性、公平性以及加锁语义等方面可能有所不同：

+ **释放优先**。写入操作释放锁时，队列同时存在读写线程，应该优先选择哪种线程，还是最先发出请求的线程？
+ **读线程插队**。锁由读线程持有，但有写线程正在等待，新到达的读线程能否立即获得访问权，还是在写线程后面等待？允许将提高并发性，却可能造成写线程发生饥饿问题。
+ **重入性**。读锁和写锁是否是可重入的？
+ **降级**。持有写锁的线程能否在不释放的情况下获得读锁？（写锁“降级”为读锁），同时不允许其他写线程修改被保护的资源。
+ **升级**。读锁优先其他读写线程升级为写锁。大多数不支持升级，容易造成死锁。

```java
public class ReadWriteMap<K, V> {
    private final Map<K, V> map;
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Lock r = lock.readLock();
    private final Lock w = lock.writeLock();

    public V put(K key, V value) {
        w.lock();
        try {
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }

    public V get(Object key) {
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }
}
```

## 构建自定义的同步工具

构建状态依赖类的最简单方法是在类库中现有状态依赖类基础上构造。如果没有提供需要的功能，可以使用底层机制来构造自己的同步机制：条件队列、显式的`Condition`对象、`AbstractQueuedSynchronizer`框架。

### 状态依赖性的管理

并发对象上状态依赖的方法有时可以在不满足前置条件时失败，但通常有更好的替代方法：等待（阻塞）前置条件变为真。

构成前置条件的状态变量必须由对象的锁保护，以便在测试前置条件时保持不变。如果前置条件不成立，则必须释放锁定，以便另一个线程可以修改对象状态。

使用轮询和休眠虽然可以解决状态依赖问题，但效率不高：

```java
public void put(V v) throws InterruptedException {
    while (true) {
        synchronized (this) {
            if (!isNull()) {
                doPut(v);
                return;
            }
        }
        Thread.sleep(SLEEP_GRANULARITY);
        // 不进入休眠，则被成为忙等待或自旋等待。时间过长将会消耗大量CPU时间
        // 进入休眠，如果状态在`sleep`后就立即发生变化，将存在不必要的休眠时间
    }
}
```

内置的条件队列可以使线程一直阻塞，直到对象进入某个线程可以继续执行的状态，并且当被阻塞的线程可以执行时再唤醒它们。这种做法更高效（当缓存状态没有发生变化时，线程醒来的次数将更少），响应性也更高（当发生特定状态变化时将立即醒来）。

#### 条件队列

“条件队列”：使得一组线程（等待线程集）能够通过某种方式来等待特定的条件变成真。传统队列的元素是数据，条件队列的元素是正在等待相关条件的线程。

正如每个Java对象都可以作为一个锁，每个对象同样可以作为一个条件队列，`Object`中的`wait`、`notify`、`notifyAll`方法构成内部条件队列的API。对象的内部锁和它的内部条件队列是相关的：要调用对象X上的任何条件队列方法，必须在X上持有锁。这是因为“等待基于状态的条件的机制”与“保持状态一致性的机制”必然是紧密绑定的。

`Object.wait`会自动释放锁，并请求操作系统挂起当前进程，从而使其他线程能够获得这个锁并修改对象的状态。当被挂起的线程醒来时，它将在返回之前重新获取锁。

```java
// 阻塞直到：not-full
public synchronized void put(V v) throws InterruptedException {
    while (isFull())
        wait();
    doPut(v);
    notifyAll();
}

// 阻塞直到:not-empty
public synchronized V take() throws InterruptedException {
    while (isEmpty())
        wait();
    V v = doTake();
    notifyAll();
    return v;
}
```

### 使用条件队列

**条件谓词**是使某个操作成为状态依赖操作的前提条件。在有界缓存中，只有当`!isEmpty()`时，`take`才能执行，否则必须等待。对`take`来说，它的条件谓词就是`!isEmpty()`。

#### 过早唤醒

内置条件队列可以与多个条件谓词一起使用，例如：`isFull()`、`isEmpty()`。当一个线程由于调用`notifyAll`而醒来时，并不意味该线程的条件谓词已变成真了。

+ 可能在调用`notifyAll`时是真的，但在重新获取锁时再次变为假（其他线程优先获取锁并修改对象状态）。
+ 可能是另一个条件谓词变成了真。

所以当线程从`wait`唤醒时，都必须再次测试条件谓词。

#### 丢失的信号

活跃性故障的一种。在调用`wait`前没有检查条件谓词，就会导致信号丢失。

#### 通知

每当在等待一个条件时，一定要确保在条件谓词变为真时通过某种方式发出通知。

+ `notify`会从这个条件队列上等待的多个线程中选择一个唤醒
+ `notifyAll`会唤醒所有这个条件队列上等待的线程

如果条件队列持有多个条件谓词，使用`notify`将是一种危险的操作，单一的通知很容易导致类似于信号丢失的问题（信号被劫持）。

使用`notify`的条件：

1. **所有等待线程的类型都相同**：只有一个条件谓词与条件队列相关。并且每个线程在从`wait`返回后将执行相同的操作。
2. **单进单出**：在条件变量上的每次通知，最多只能唤醒一个线程来执行。

条件通知：

```java
public synchronized void put(V v) throws InterruptedException {
    while (isFull())
        wait();
    boolean wasEmpty = isEmpty();
    doPut(v);
    // 由空变为非空
    if (wasEmpty())
        notifyAll();
}
```

单次通知和条件通知都属于优化措施。

### 显式的Condition对象

内置条件队列存在一些缺陷。每个内置锁都只能有一个相关联的条件队列，而条件队列可能有多个条件谓词。`notifyAll`无法单一唤醒某一类型的线程。

一个`Condition`和一个`Lock`关联在一起，如同一个条件队列关联一个内置锁。`Condition`比内置条件队列提供更丰富的功能：每个锁存在多个等待、条件等待可以是可中断或不可中断的、基于时限的等待，以及公平的或非公平的队列操作。

```java
protected final Lock lock = new ReentrantLock();
private final Condition notFull = lock.newCondition();
private final Condition notEmpty = lock.newCondition();
private final T[] items = (T[]) new Object[BUFFER_SIZE];
private int tail, head, count;

public void put(T x) throws InterruptedException {
    lock.lock();
    try {
        while (count == items.length)
            notFull.await();
        items[tail] = x;
        if (++tail == items.length)
            tail = 0;
        ++count;
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
}

public T take() throws InterruptedException {
    lock.lock();
    try {
        while (count == 0)
            notEmpty.await();
        T x = items[head];
        items[head] = null;
        if (++head == items.length)
            head = 0
        --count;
        notFull.signal();
        return x;
    } finally {
        lock.unlock();
    }
}
```

`signal`比`signalAll`更高效，它能极大地减少在每次缓存操作中发生上下文切换与锁请求的次数。

### AbstractQueuedSynchronizer

基于AQS构建的工具类：`ReentrantLock`, `Semaphore`, `CountDownLatch`, `ReentrantReadWriteLock`, `SynchronousQueue`, `FutureTask`。

AQS解决了实现同步器涉及的大量细节问题：FIFO队列（公平锁）。

AQS负责管理同步器中的状态，一个整数状态信息。可通过`getState`，`setState`以及`compareAndSetState`等`protected`类型方法进行操作。

+ `ReentrantLock`：所有者线程已经重复获取该锁的次数
+ `Semaphore`：剩余的许可数量
+ `FutureTask`：任务的状态（尚未开始、正在运行、已完成、已取消）

## 原子变量与非阻塞同步机制

非阻塞算法：用底层的原子机器指令代替锁来确保数据在并发访问中的一致性。使多个线程在竞争相同的数据时不会发生阻塞。不存在死锁、活跃性问题。

原子变量提供了与`volatile`类型相同的内存语义，此外还支持原子的更新操作。

### 锁的劣势

多个线程同时请求锁，一些线程将被挂起（借助os功能）并稍后运行。恢复执行时，必须等待其他线程执行完它们的时间片，才能被调度执行。挂起和恢复等过程存在很大的开销。

`volatile`是一种更轻量级的同步机制，但它只提供可见性，没有原子性。

其他缺点：线程在等待锁时，不能做其他事情。如果一个线程在持有锁的情况下被延迟执行（缺页错误、调度延迟），所有需要这个锁的线程都无法执行下去。

### 硬件对并发的支持

独占锁是一项悲观技术。对于细粒度操作，有一种更高效的乐观方法，可以在不发生干扰的情况下完成更新操作。

几乎所有的现代处理器中都包含了某种形式的原子读-改-写指令：比较并交换（Compare-and-Swap），关联加载/条件存储（Load-Linked/Store-Conditional）。

#### 比较并交换

CAS包含3个操作数-需要读写的内存位置V、进行比较的值A和拟写入的新值B。当且仅当V的值等于A时，才会通过原子方式用新值B来更新V的值，否则不会执行任何操作。结果始终返回V原有的值。

多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新成功，其他线程都将失败，但不会被挂起。失败后可以再次尝试或执行一些恢复操作，或不执行任何操作。大大减少活跃性风险。

```java
private SimulatedCAS value;

public int getValue() {
    return value.get();
}

public int increment() {
    int v;
    do {
        v = value.get();
    } while (v != value.compareAndSwap(v, v+1));
    return v+1;
}
```

CAS的主要缺点是，调用者处理竞争问题（重试、回退、放弃），而锁能自动处理竞争问题（线程获得锁之前将一直阻塞）。

大多数处理器上，在无竞争的锁获取和释放的“快速代码路径”上的开销，大约是CAS开销的两倍。

### 原子变量类

原子变量类相当于一种泛化的`volatile`变量，能够支持原子的和有条件的读-改-写操作。

标量类：`AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference`。原子数组类中的元素可以实现原子更新。为数组的元素提供了`volatile`类型的访问语义

在高度竞争的情况下，锁的性能将超过原子变量的性能。而在更真实的竞争情况下（中低程度竞争），原子变量的性能超过锁的性能。锁在发生竞争时会挂起线程，从而降低CPU的使用率和共享内存总线上的同步通信量。原子变量在遇到竞争时立即重试（消耗CPU），在激烈竞争环境导致更多的竞争。

#### ABA问题

在CAS操作中将判断“V的值是否仍然为A？”，有时候还需要知道“自从上次看到V的值为A以来，这个值是否发生了变化？”

相对简单的解决方案：不是更新某个引用的值，而是更新两个值：引用、版本号。`AtomicStampedReference`, `AtomicMarkableReference

## Java内存模型

处理器可以采用乱序或并行等方式来执行指令；缓存可能会改变将写入变量提交到主内存的次序；保存在处理器本地缓存中的值，对于其他处理器是不可见的。Java语言规范要求JVM在线程中维护一种类似串行的语义：只要程序的最终结果与在严格串行环境中执行的结果相同，那么上述所有操作都是允许的。

多线程环境中，维护程序的串行性将导致很大的性能开销。只有要共享数据时，才必须协调它们之间的操作。

JMM为程序中所有的操作定义了一个偏序关系，称为Happens-Before。

+ **程序顺序规则**
+ **监视器锁规则**
+ **volatile变量规则**
+ **线程启动规则**
+ **线程借宿规则**
+ **中断规则**
+ **终结器规则**
+ **传递性**

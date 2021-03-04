# Go 的数据类型 rune 和 byte

    byte 是 uint8 的别名，它表示的是 ACSII 表中的一个字符。
    rune 是 int32 的别名，几乎在所有方面等同于 int32，它表示的是一个 Unicode 字符。

    因为 uint8 和 uint32，直观上让人以为这是一个数值，但是实际上，它也可以表示一个字符，
    所以为了消除这种直观错觉，就诞生了 byte 和 rune 这两个别名类型。

# slice 的底层实现

    slice 本身并不是动态数组或者数组指针。它内部实现的数据结构通过指针引用底层数组，设定相关属性将数据读写操作限定在指定的区域内。
    ```
    type slice struct {
        array unsafe.Pointer
        len   int
        cap   int
    }
    ```
    slice 的结构体由3部分构成，Pointer 是指向一个数组的指针，len 代表当前切片的长度，cap 是当前切片的容量。cap 总是大于等于 len 的。

# slice 扩容策略

    1. 首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）
    2. 否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap）
    3. 否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的 1/4，
       即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
    4. 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）

    注意：扩容扩大的容量都是针对原来的容量而言的，而不是针对原来数组的长度而言的。

    slice 扩容后是新数组 or 老数组？
        1. 由于原数组还有容量可以扩容，所以执行 append() 操作以后，会在原数组上直接操作，
           所以这种情况下，扩容以后的数组还是指向原来的数组。
        2. 如果原来数组的容量已经达到了最大值，再想扩容，Go 默认会先开一片内存区域，把原来的值拷贝过来，然后再执行 append() 操作。
           这种情况丝毫不影响原数组。

# Map 使用什么方法解决 hash 冲突？

    拉链法

# Map 扩容的过程

    参考：https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/

    哈希在存储元素过多时会触发扩容操作，每次都会将桶的数量翻倍，扩容过程不是原子的，而是通过 runtime.growWork 增量触发的，
    在扩容期间访问哈希表时会使用旧桶，向哈希表写入数据时会触发旧桶元素的分流。

# 使用运算符"+"连接字符串会有性能问题吗？

    使用运算符"+"连接字符串每次都会产生新的字符串，涉及到拷贝，
    并且会产生很多临时的无用的字符串，给 gc 带来额外的负担，所以性能比较差。

# 怎么高效拼接字符串？

    使用 strings.Builder，因为是 strings.Builder 以倍数申请内存，并且直接将底层的 []byte 转换成字符串类型返回。

# Go 在函数参数传递的时候是值传递还是引用传递？

    无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝。

    整型和数组类型在调用函数时会对内容进行拷贝，不影响原值。如果当前数组的大小非常的大，这种传值的方式会对性能造成比较大的影响。
    传递结构体时，会拷贝结构体中的全部内容。
    传递结构体指针时，会拷贝结构体指针。

# 反射

    Go 语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法，但是在编译时并不知道这些变量的具体类型，这称为反射机制。

    三大法则：
        1. 从 interface{} 变量可以反射出反射对象；
        2. 从反射对象可以获取 interface{} 变量；
        3. 要修改反射对象，其值必须可设置。

# 下面的 for range 代码是死循环么？

    ```
    func main() {
        v := []int{1, 2, 3}
        for i := range v {
            v = append(v, i)
        }
    }
    ```

    不会，在遍历之前会对切片复制，遍历的次数不会随着切片的变化而变化

# 对大数组 for range 遍历有什么问题？

    ```
     //假设值都为1，这里只赋值3个
    var arr = [102400]int{1, 1, 1} 
    for i, n := range arr {
        //just ignore i and n for simplify the example
        _ = i 
        _ = n 
    }
    ```

    遍历前会对数组进行拷贝，浪费内存并且影响性能，
    可以对数组取地址遍历 for i, n := range &arr，或者对数组做切片引用 for i, n := range arr[:]

# for range 对遍历的元素取地址加到另外一个切片后有什么问题？

    ```
    func main() {
        arr := []int{1, 2, 3}
        newArr := []*int{}
        for _, v := range arr {
            newArr = append(newArr, &v)
        }
        for _, v := range newArr {
            fmt.Println(*v)
        }
    }

    // 输出 3 3 3
    ```

    在 for range 中，变量 v 每次迭代的值都是赋值给 v，该变量的内存地址始终未变，
    这样把它的地址追加到新的切片中，该切片保存的都是同一个地址。
    可以在 newArr = append(newArr, &v) 前面加上 v := v，可以通过下标取原切片的值

# for range 迭代修改变量问题

    ```
    package main

    import (
        "fmt"
    )

    type user struct {
        name string
        age uint64
    }

    func main()  {
        u := []user{
            {"asong",23},
            {"song",19},
            {"asong2020",18},
        }
        for _,v := range u{
            if v.age != 18{
                v.age = 20
            }
        }
        fmt.Println(u)
    }

    // [{asong 23} {song 19} {asong2020 18}]
    ```

    v 变量是拷贝切片中的数据，修改拷贝数据不会对原切片有影响

# for-range 里 goroutine 闭包捕获的代码问题

    ```
    for _, val := range values {
        go func() {
            fmt.Println(val)
        }()
    }
    ```
    或
    ```
    for _, val := range values {
        go val.MyMethod()
    }

    func (v *val) MyMethod() {
            fmt.Println(v)
    }
    ```

    大概率都会只打印最后的值，goroutine 很可能会在循环过后才启动，每次 goroutine 都是获取同样的变量 val
    可以改成

    ```
    for _, val := range values {
        go func(val interface{}) {
            fmt.Println(val)
        }(val)
    }
    ```
    或
    ```
    for i := range valslice {
        val := valslice[i]
        go func() {
            fmt.Println(val)
        }()
    }
    ```

    注意：如果不在 goroutine 中使用闭包，不会出现这样的问题
    ```
    for i := 1; i <= 5; i++ {
        func() {
            fmt.Println(i)
        }()
    }

    // 1 2 3 4 5
    ```

# select 在遇到多个 Channel 同时响应时，怎样执行？

    随机执行一种情况（避免饥饿问题，保证公平性）

# select 在多个文件或者 Channel 状态改变之前会怎么样？

    如果 select 控制结构中不包含 default 语句，select 会一直阻塞当前线程或者 Goroutine。
    如果 select 控制结构中包含 default 语句，那么这个 select 语句在执行时会遇到以下两种情况：
        1. 当存在可以收发的 Channel 时，直接处理该 Channel 对应的 case；
        2. 当不存在可以收发的 Channel 时，执行 default 中的语句（可以用这种方式实现非阻塞）；

# defer 的执行顺序？

    每次遇到 defer 相当于放入栈中（底层是用链表模拟栈），最终取出执行

# defer 的调用时机？

    defer 传入的函数不是在退出代码块的作用域时执行的，它只会在当前函数和方法返回之前被调用。

# defer 预计算参数

    ```
    func main() {
        startedAt := time.Now()
        defer fmt.Println(time.Since(startedAt))
        
        time.Sleep(time.Second)
    }

    // 0s
    ```

    defer 关键字会立刻拷贝函数中引用的外部参数，所以 time.Since(startedAt) 的结果不是在 main 函数退出之前计算的，
    而是在 defer 关键字调用时计算的，最终导致上述代码输出 0s。

    解决方法：
    ```
    func main() {
        startedAt := time.Now()
        defer func() { fmt.Println(time.Since(startedAt)) }()
        
        time.Sleep(time.Second)
    }

    // 1s
    ```

    向 defer 关键字传入匿名函数，虽然调用 defer 关键字时也使用值传递，但是拷贝的是函数指针，可以打印出符合预期的结果。

# recover 只有在 defer 中调用才会生效

# make 和 new 的区别

    new:
        根据传入的类型分配 zeroed（零值）空间并返回指向这片内存空间的指针。
        ```
        i := new(int)

        var v int
        i := &v

        // 两种不同初始化方法是等价的，它们都会创建一个指向 int 零值的指针
        ```
    make:
        只能初始化 slice，map，channel，不返回指针
        ```
        // Using make() to initialize a map.
        m := make(map[string]bool, 0)

        // Using a composite literal to initialize a map.
        m := map[string]bool{}

        // 两种方法是等价的
        ```
        注意 new 返回的是零值，而 slice，map，channel 的零值是 nil，所以 new 返回的上述三种类型是不能直接使用的

# context 包

    用途：
        是在不同 Goroutine 之间同步请求特定数据、取消信号以及处理请求的截止日期。
        可能会创建多个 Goroutine 来处理一次请求，不使用 context，每个 Goroutine 可能无限执行下去。

    context.Context 接口需要实现的方法：
        Deadline
            返回 context.Context 被取消的时间，也就是完成工作的截止日期
        Done
            返回一个 Channel，这个 Channel 会在当前工作完成或者上下文被取消之后关闭，
            多次调用 Done 方法会返回同一个 Channel
        Err
            返回 context.Context 结束的原因，它只会在 Done 返回的 Channel 被关闭时才会返回非空的值
            如果 context.Context 被取消，会返回 Canceled 错误；
            如果 context.Context 超时，会返回 DeadlineExceeded 错误；
        Value
            从 context.Context 中获取键对应的值，对于同一个上下文来说，多次调用 Value 并传入相同的 Key 会返回相同的结果，
            该方法可以用来传递请求特定的数据

    WithCancel 函数
        func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
        传递一个父 Context 作为参数，返回子 Context，以及一个取消函数用来取消 Context。
    WithDeadline 函数
        func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
        和 WithCancel 差不多，它会多传递一个截止时间参数，意味着到了这个时间点，会自动取消 Context，
        当然我们也可以不等到这个时候，可以提前通过取消函数进行取消。
    WithTimeout 函数
        func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
        和 WithDeadline 基本上一样，这个表示是超时自动取消，是多少时间后自动取消 Context 的意思。
    WithValue 函数
        func WithValue(parent Context, key, val interface{}) Context
        和取消 Context 无关，它是为了生成一个绑定了一个键值对数据的 Context，这个绑定的数据可以通过 Context.Value 方法访问到。

    Context 是线程安全的吗？
        是，可以放心的在多个 goroutine 中传递。同一个 Context 可以传给使用其的多个 goroutine，
        且 Context 可被多个 goroutine 同时安全访问。
        因为 context 本身是不可变的（Value 也一定应该是线程安全的）。

# sync.Map

    参考：
        [由浅入深聊聊Golang的sync.Map](https://juejin.cn/post/6844903895227957262)

    map 在并发情况下，只读是线程安全的，同时写线程不安全。
    可以通过 sync.RWMutex 自己实现并发安全的 map

    官方实现的 sync.Map 的核心数据结构:
    ```
    type Map struct {
        mu Mutex
        read atomic.Value // readOnly
        dirty map[interface{}]*entry
        misses int
    }
    ```
    mu 加锁作用。保护后文的 dirty 字段
    read 存读的数据。因为是 atomic.Value 类型，只读，所以并发是安全的。实际存的是 readOnly 的数据结构。
    misses 计数作用。每次从 read 中读失败，则计数 +1。
    dirty 包含最新写入的数据。当 misses 计数达到一定值，将其赋值给 read。
    写：直写 dirty。
    读：先读 read，没有再读 dirty。

    优缺点：
        优点：通过读写分离，降低锁时间来提高效率。
        缺点：不适用于大量写的场景，这样会导致 read map 读不到数据而进一步加锁读取，
              同时 dirty map 也会一直晋升为 read map，整体性能较差。
    适用场景：大量读，少量写

# sync.pool

    参考：https://zhuanlan.zhihu.com/p/133638023
          https://juejin.cn/post/6844903903046320136#heading-14

    频繁地分配、回收内存会给 GC 带来一定的负担，严重的时候会引起 CPU 的毛刺，sync.Pool 可以将暂时不用的对象缓存起来，
    待下次需要的时候直接使用，不用再次经过内存分配，复用对象的内存，减轻 GC 的压力，提升系统的性能。
    ```
    type Pool struct {
        // noCopy，防止当前类型被 copy
        noCopy noCopy

        // [P]poolLocal 数组指针
        local     unsafe.Pointer
        // 数组大小
        localSize uintptr

        // 选填的自定义函数，缓冲池无数据的时候会调用，不设置默认返回 nil
        New func() interface{} //新建对象函数
    }

    type poolLocalInternal struct {
        // 私有缓存区
        private interface{}
        // 公共缓存区
        shared  []interface{}
        // 锁
        Mutex
    }

    type poolLocal struct {
        // 每个 P 对应的 pool
        poolLocalInternal

        // 防止"false sharing/伪共享"
        pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
    }
    ```
    GMP 模型中的每个 P 对应一个 poolLocal
    put：优先放入 private 空间，后面再放入 shared 空间
    get：优先从 private 空间拿，再加锁从 shared 空间拿，还没有再从其他的 PoolLocal 的 shared 空间拿，
         还没有就直接 new 一个返回。

    sync.Pool的特性
        1. 无大小限制。
        2. 自动清理，每次 GC 前会清掉 Pool 里的所有对象。所以不适用于做连接池。
        3. 每个 P 都会有一个本地的 poolLocal，Get 和 Put 优先在当前 P 的本地 poolLocal 操作。其次再进行跨 P 操作。
        4. 所以 Pool 的最大个数是 runtime.GOMAXPROCS(0)。

    pool 是永久保存的吗？
        会进行清理，时间就是两次 GC 间隔的时间。
        sync.Pool 只是对对象的复用，不可以当做对象保存（比如连接池），因为 Pool 中的对象随时可能被 GC 了。

    为什么获取 shared 要加锁，而 private 不用？
        每个 P 都分配一个 localPool，在同一个 P 下面只会有一个 Gouroutine 在跑，
        所以这里的 private，在同一时间就只可能被一个 Gouroutine 获取到。
        而 shared 就不一样了，有可能被其他的 P 给获取走，在同一时间就只可能被多个 Gouroutine 获取到，
        为了保证数据竞争，必须加一个锁来保证只会被一个 G 拿走。

    为什么这样设计？
        Goroutine 能同一时刻在并行的数量有限，是由 runtime.GOMAXPROCS(0)设置的，
        这里的 Pool 将数据与 P 进行绑定了，分散在了各个真正并行的线程中，
        每个线程优先从自己的 poolLocal 中获取数据，很大程度上降低了锁竞争。

# sync 包

    包括常见的 sync.Mutex、sync.RWMutex、sync.WaitGroup、sync.Once、sync.Cond 和 sync.Map、sync.pool 等等
    sync.Map、sync.pool 上面介绍过

    sync.Mutex
        ```
        type Mutex struct {
            state int32
            sema  uint32
        }
        ```
        state 表示当前互斥锁的状态，而 sema 是用于控制锁状态的信号量。
        state 中最低三位分别表示 mutexLocked、mutexWoken 和 mutexStarving，剩下的位置为 waitersCount。
                mutexLocked — 表示互斥锁的锁定状态；
                mutexWoken — 表示从正常模式被从唤醒；
                mutexStarving — 当前的互斥锁进入饥饿状态；
                waitersCount — 当前互斥锁上等待的 Goroutine 个数；

        sync.Mutex 有两种模式 — 正常模式和饥饿模式。
            在正常模式下，锁的等待者会按照先进先出的顺序获取锁。
            但是刚被唤起的 Goroutine 与新创建的 Goroutine 竞争时，大概率会获取不到锁，
            为了减少这种情况的出现，一旦 Goroutine 超过 1ms 没有获取到锁，它就会将当前互斥锁切换饥饿模式，
            防止部分 Goroutine 被『饿死』。

        加锁：
            1. 通过原子操作判断
               当锁的状态是 0 时，将 mutexLocked 位置成 1，返回；
               如果互斥锁的状态不是 0 时就会调用 sync.Mutex.lockSlow 尝试通过自旋（Spinnig）等方式等待锁的释放，
               该方法的主体是一个非常大 for 循环，这里将它分成几个部分介绍获取锁的过程：
            2. 判断当前 Goroutine 能否进入自旋；
            3. 通过自旋等待互斥锁的释放；
            4. 计算互斥锁的最新状态；
            5. 通过原子操作更新互斥锁的状态并获取锁；

        解锁：
            1. 先校验锁状态的合法性，如果当前互斥锁已经被解锁过了会直接抛出异常"sync: unlock of unlocked mutex"中止当前程序。
            2. 在正常模式下
                1. 如果互斥锁不存在等待者或者互斥锁的 mutexLocked、mutexStarving、mutexWoken 状态不都为 0，
                   那么当前方法可以直接返回，不需要唤醒其他等待者；
                2. 如果互斥锁存在等待者，会通过 sync.runtime_Semrelease 唤醒等待者并移交锁的所有权；
               在饥饿模式下，上述代码会直接调用 sync.runtime_Semrelease 将当前锁交给下一个正在尝试获取锁的等待者，
               等待者被唤醒后会得到锁，在这时互斥锁还不会退出饥饿状态；

    sync.RWMutex
        读写互斥锁 sync.RWMutex 是细粒度的互斥锁，它不限制资源的并发读，但是读写、写写操作无法并行执行。

        ```
        type RWMutex struct {
            w           Mutex
            writerSem   uint32
            readerSem   uint32
            readerCount int32
            readerWait  int32
        }
        ```
        w — 复用互斥锁提供的能力；
        writerSem 和 readerSem — 分别用于写等待读和读等待写的信号量：
        readerCount 存储了当前正在执行的读操作数量；
        readerWait 表示当写操作被阻塞时等待的读操作个数；

    sync.WaitGroup
        可以等待一组 Goroutine 的返回。

    sync.Once
        可以保证在 Go 程序运行期间的某段代码只会执行一次。

    sync.Cond
        可以让一组的 Goroutine 都在满足特定条件时被唤醒。

# 计时器

    所有的计时器都以最小四叉堆的形式存储在处理器 runtime.p 中。

    计时器什么时间触发？
        1. 调度器调度时会检查处理器中的计时器是否准备就绪；
        2. 系统监控会检查是否有未执行的到期计时器；

# 系统监控

    当 Go 语言程序启动时，会创建新的独立线程启动系统监控。
    主要有以下功能：
        1. 检查死锁
        2. 运行计时器 — 获取下一个需要被触发的计时器；
        3. 轮询网络 — 获取需要处理的到期文件描述符；
        4. 抢占处理器 — 抢占运行时间较长的或者处于系统调用的 Goroutine；
        5. 垃圾回收 — 在满足条件时触发垃圾收集回收内存；

# Channel

    发送和接收数据的顺序：
        带缓冲区和不带缓冲区的 Channel 都会遵循先入先出发送和接收数据

    ```
    type hchan struct {
        qcount   uint
        dataqsiz uint
        buf      unsafe.Pointer
        elemsize uint16
        closed   uint32
        elemtype *_type
        sendx    uint
        recvx    uint
        recvq    waitq
        sendq    waitq

        lock mutex
    }
    ```
    qcount — Channel 中的元素个数；
    dataqsiz — Channel 中的循环队列的长度；
    buf — Channel 的缓冲区数据指针（只针对有缓冲的 channel，指向底层循环数组）；
    sendx — Channel 的发送操作处理到的位置；
    recvx — Channel 的接收操作处理到的位置；
    elemsize 和 elemtype 分别表示当前 Channel 能够收发的元素类型和大小；
    sendq 和 recvq 存储了当前 Channel 由于缓冲区空间不足而阻塞的 Goroutine 列表，这些等待队列使用双向链表表示；

    创建管道：
        Go 语言中所有 Channel 的创建都会使用 make 关键字。
        如果不向 make 传递表示缓冲区大小的参数，那么就会设置一个默认值 0，也就是当前的 Channel 不存在缓冲区，
        否则就是有缓冲区的 Channel。

    接收和发送数据都需要先获取锁。

<p align='center'>
    <img src='./images/操作 channel 结果.jpg'>
</p>

# Goroutine 原理（调度器）

    参考：
        1. [Golang 的 goroutine 是如何实现的？ - 凌霄Leon的回答 - 知乎](https://www.zhihu.com/question/20862617/answer/131341519)
        2. [Golang 的 goroutine 是如何实现的？ - ZeaTalk的回答 - 知乎](https://www.zhihu.com/question/20862617/answer/710435704)
        3. [[典藏版] Golang 调度器 GMP 原理与调度全分析](https://learnku.com/articles/41728)
        4. [Go 语言设计与实现 6.5 调度器](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#%E7%BA%BF%E7%A8%8B%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
        5. [Go语言为何不使用异步文件IO和PoolDesc？ - 知乎](https://www.zhihu.com/question/54743395)
        6. [Go netpoller 原生网络模型之源码全面揭秘](https://strikefreedom.top/go-netpoll-io-multiplexing-reactor)
        7. [为什么要使用 Go 语言？Go 语言的优势在哪里？ - 腾讯技术工程的回答 - 知乎](https://www.zhihu.com/question/21409296/answer/1040884859)

    版本演进：
        （0.x） 单线程调度器：
            只包含 40 多行代码；
            程序中只能存在一个活跃线程，由 G-M 模型组成；
        （1.0） 多线程调度器
            允许运行多线程的程序；
            全局锁导致竞争严重；
        （1.1） 任务窃取调度器
            引入了处理器 P，构成了目前的 G-M-P 模型；
            在处理器 P 的基础上实现了基于工作窃取的调度器；
            在某些情况下，Goroutine 不会让出线程，进而造成饥饿问题；
            时间过长的垃圾回收（Stop-the-world，STW）会导致程序长时间无法工作；
        （1.2 ~ 至今） 抢占式调度器
            （1.2 ~ 1.13） 基于协作的抢占式调度器
                通过编译器在函数调用时插入抢占检查指令，在函数调用时检查当前 Goroutine 是否发起了抢占请求；
                Goroutine 可能会因为垃圾回收和循环（没有函数调用）长时间占用资源导致程序暂停；
            （1.14 ~ 至今） 基于信号的抢占式调度器
                实现基于信号的真抢占式调度；
                首先注册绑定 SIGURG 信号及处理方法，在触发垃圾回收的栈扫描时，调用函数挂起 Goroutine，
                并向 M 发送信号，M 收到信号后，会让当前 Goroutine 陷入休眠继续执行其他的 Goroutine，
                抢占的时间点不够多（只有触发垃圾回收的栈扫描时，因为这是一个可以抢占的安全点），还不能覆盖全部的边缘情况；
        （提案） 非均匀存储访问调度器
            对运行时的各种资源进行分区；
            实现非常复杂，到今天还没有提上日程；

    G-P-M 模型：
        G — 表示 Goroutine，它是一个待执行的任务；
        M — 表示操作系统的线程，它由操作系统的调度器调度和管理；
        P — 表示处理器，它可以被看做运行在线程上的本地调度器
        全局队列（Global Queue）：存放等待运行的 G。
        P 的本地队列：同全局队列类似，存放的也是等待运行的G，存的数量有限，不超过256个。
                      新建 G 时，G 优先加入到 P 的本地队列，如果队列满了，则会把本地队列中一半的 G 移动到全局队列。
        P列表：所有的 P 都在程序启动时创建，并保存在数组中，最多有 GOMAXPROCS 个。
        M：线程想运行任务就得获取 P，从 P 的本地队列获取 G，
           P 队列为空时，M 也会尝试从全局队列拿一批 G放到 P 的本地队列，
           或从其他 P 的本地队列偷一半放到自己 P 的本地队列。
           M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。
        Goroutine 调度器和 OS 调度器是通过 M 结合起来的，
        每个 M 都代表了1个内核线程，OS 调度器负责把内核线程分配到 CPU 的核上执行。

<p align='center'>
    <img src='./images/G-P-M 模型的结构.jpg'>
</p>

    有关 P 和 M 的个数问题：
        P 的数量：
            1. 由启动时环境变量 $GOMAXPROCS 或者是由 runtime 的方法 GOMAXPROCS() 决定。
            2. 这意味着在程序执行的任意时刻都只有 $GOMAXPROCS 个 goroutine 在同时运行。
        M 的数量：
            1. go 语言本身的限制：go 程序启动时，会设置 M 的最大数量，默认 10000。
               但是内核很难支持这么多的线程数，所以这个限制可以忽略。
            2. runtime/debug 中的 SetMaxThreads 函数，可以设置 M 的最大数量。
        M 与 P 的数量没有绝对关系，一个 M 阻塞，P 就会去创建或者切换另一个 M，
        所以，即使 P 的默认数量是 1，也有可能会创建很多个 M 出来。

    调度循环：
        调度器启动之后，Go 语言运行时会初始化线程并调用 runtime.schedule 进入调度循环：

        runtime.schedule 函数会从下面几个地方查找待执行的 Goroutine：
            1. 为了保证公平，当全局运行队列中有待执行的 Goroutine 时，
               保证有 1/61 的几率会从全局的运行队列中查找对应的 Goroutine
               （全局运行队列是所有工作线程都可以访问的，所以在访问它之前需要加锁）；
            2. 从处理器本地的运行队列中查找待执行的 Goroutine；
            3. 如果前两种方法都没有找到 Goroutine，会通过 runtime.findrunnable 进行阻塞地查找 Goroutine；

        runtime.findrunnable：
            1. 从本地运行队列、全局运行队列中查找；
            2. 从网络轮询器中查找是否有 Goroutine 等待运行；
            3. 通过 runtime.runqsteal 尝试从其他随机的处理器中窃取待运行的 Goroutine；
            因为函数的实现过于复杂，上述的执行过程是经过简化的，
            总而言之，当前函数一定会返回一个可执行的 Goroutine，
            如果当前不存在就会通过轮询网络的方式直到返回可用的 Goroutine，也就是线程自旋（最大数量为 GOMAXPROCS）。

        接下来由 runtime.execute 执行获取的 Goroutine，
        做好准备工作后，它会通过 runtime.gogo 将 Goroutine 调度到当前线程上。
        当 Goroutine 中运行的函数返回时，最终在当前线程的 g0 的栈上调用 runtime.goexit0 函数，
        该函数会将 Goroutine 转换会 _Gdead 状态、清理其中的字段、
        移除 Goroutine 和线程的关联并把 Goroutine 重新加入处理器的 Goroutine 空闲列表 gFree：

        在最后 runtime.goexit0 会重新调用 runtime.schedule 触发新一轮的 Goroutine 调度，

        Go 语言中的运行时调度循环会从 runtime.schedule 开始，最终又回到 runtime.schedule，
        可以认为调度循环永远都不会返回。

<p align='center'>
    <img src='./images/golang-scheduler-loop.png'>
</p>

        调度时机：
            1. 系统调用
            2. time 定时类操作
            3. 使用关键字 go
            4. atomic，mutex，channel 操作等会使 Goroutine 阻塞
            5. GC
            6. 主动调用 runtime.Gosched()
            7. 系统监控
            等等

        阻塞时怎样调度：
            1. 由于原子、互斥量或通道操作调用导致 Goroutine 阻塞，调度器将把当前阻塞的 Goroutine 切换出去，
               重新调度 LRQ（本地队列） 上的其他 Goroutine；
            2. 由于网络请求和 IO 操作（LInux 不包括文件IO，详见参考 5）导致 Goroutine 阻塞。
               Go 程序提供了网络轮询器（NetPoller）来处理网络请求和 IO 操作的问题，
               其后台通过 kqueue（MacOS），epoll（Linux）或 iocp（Windows）来实现 IO 多路复用。
               通过使用 NetPoller 进行网络系统调用，调度器可以防止 Goroutine 在进行这些系统调用时阻塞 M。
               这可以让 M 执行 P 的 LRQ 中其他的 Goroutines，而不需要创建新的 M。
               执行网络系统调用不需要额外的 M，网络轮询器使用系统线程，它时刻处理一个有效的事件循环，有助于减少操作系统上的调度负载。
               用户层眼中看到的 Goroutine 中的“block socket”，实现了 goroutine-per-connection 简单的网络编程模式。
               实际上是通过 Go runtime 中的 netpoller 通过 Non-block socket + I/O 多路复用机制“模拟”出来的。
            3. 当调用一些系统方法的时候（如文件 I/O 等，详见参考 5），如果系统方法调用的时候发生阻塞，
               这种情况下，网络轮询器（NetPoller）无法使用，而进行系统调用的 G1 将阻塞当前 M1，此时 P 会和 M1 解绑，
               调度器引入其它 M 来服务 M1 的P。
            4. 如果在 Goroutine 去执行一个 sleep 操作，导致 M 被阻塞了。
               Go 程序后台有一个监控线程 sysmon，它监控那些长时间运行的 G 任务然后设置可以强占的标识符，
               别的 Goroutine 就可以抢先进来执行。

# Go 的垃圾回收机制

    todo

    参考：
        1. [Go 语言设计与实现 7.2 垃圾收集器](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/#722-%E6%BC%94%E8%BF%9B%E8%BF%87%E7%A8%8B)
        2. [Golang三色标记+混合写屏障GC模式全分析](https://zhuanlan.zhihu.com/p/334999060)

        * Go V1.3之前标记-清除(mark and sweep)算法
        * Go V1.5的三色并发标记法
        * Go V1.8混合写屏障机制

    当前Golang使用的垃圾回收机制是三色标记发配合写屏障和辅助GC，三色标记法是标记-清除法的一种增强版本。

    标记-清除法需要程序暂停，即 STW(stop the world)
    三色并发需要遵循三色不变性，屏障技术就是在并发或者增量标记过程中保证三色不变性的重要技术

    垃圾回收中的写屏障可以理解为编译器在写操作时特意插入的一段代码，对应的还有读屏障。
    因为读屏障需要在读操作中加入代码片段，对用户程序的性能影响很大，所以编程语言往往都会采用写屏障保证三色不变性。

    为什么需要写屏障，很简单，对于和用户程序并发运行的垃圾回收算法，用户程序会一直修改内存，所以需要记录下来。

    Golang 1.7 之前的写屏障使用的经典的 Dijkstra-style insertion write barrier， STW 的主要耗时就在 stack re-scan 的过程。
    自 1.8 之后采用一种混合的写屏障方式 （Yuasa-style deletion write barrier 和 Dijkstra-style insertion write barrier）来避免 re-scan

# 逃逸分析

    参考：
        [Go 逃逸分析](https://geektutu.com/post/hpg-escape-analysis.html)
        [Go变量逃逸分析](https://www.cnblogs.com/itbsl/p/10476674.html)

    逃逸分析决定一个变量是分配在堆上还是分配在栈上。
    栈内存分配则会非常快，并且栈上内存由在函数返回时自动回收，可以能减小 gc 压力。
    如果变量分配到堆上，堆不像栈可以自动清理。它会引起 Go 频繁地进行垃圾回收，而垃圾回收会占用比较大的系统开销。

    会发上逃逸的情况：
        1. 指针逃逸
           在函数中创建了一个对象，返回了这个对象的指针。
           这种情况下，函数虽然退出了，但是因为指针的存在，对象的内存不能随着函数结束而回收，因此只能分配在堆上。
        2. interface{} 动态类型逃逸
        3. 栈空间不足
        4. 闭包

# 内联优化

    参考：
        [Go 性能调优之 —— 编译优化](https://segmentfault.com/a/1190000016354799)
        [详解Go内联优化](https://segmentfault.com/a/1190000039146279)

    函数调用是存在一些固定开销的，例如维护帧指针寄存器 BP、栈溢出检测等。
    因此，对于一些代码行比较少的函数，编译器倾向于将它们在编译期展开从而消除函数调用，这种行为就是内联。

    内联只对叶子函数有效，但是严重的内联会使得堆栈信息更加难以跟踪。

# Go 和 Python 里面的协程有什么区别？

    Python 协程的特点：
        1. 从协程：线程的对应方式来看是 N:1 模式，多个协程在一个线程中切换。在IO密集时切换效率高，但没有用到多核。
        2. 协程间完全同步，不会并行。不需要考虑数据安全。
        3. 属于协作式任务处理，程序需要主动交出控制权，宿主才能获得控制权并将控制权交给其他 coroutine。
           如果开发者无意间或者故意让应用程序长时间占用 CPU，表现出来的效果就是计算机很容易失去响应或者死机。

    Go 协程的特点：
        1. 从协程：线程的对应方式来看是 M:N 模式，多个协程在多个线程上切换，既可以用到多核，又可以减少切换开销。
        2. 协程间需要保证数据安全，比如通过 channel 或锁。
        3. 协程间不完全同步，可以并行运行，具体要看 channel 的设计。
        4. 属于抢占式任务处理，如果发现一个应用程序长时间大量地占用 CPU，那么用户有权终止这个任务。

# 为什么说使用通信共享内存，而不是使用共享内存通信？

    参考：
        [为什么使用通信来共享内存](https://draveness.me/whys-the-design-communication-shared-memory/)

    1. （抽象层级）首先，使用发送消息来同步信息相比于直接使用共享内存和互斥锁是一种更高级的抽象，
       使用更高级的抽象能够为我们在程序设计上提供更好的封装，让程序的逻辑更加清晰；
    2. （解耦合）其次，消息发送在解耦方面与共享内存相比也有一定优势，可以将线程的职责分成生产者和消费者，
       并通过消息传递的方式将它们解耦，不需要再依赖共享内存；
    3. （避免线程竞争）最后，Go 语言选择消息发送的方式，通过保证同一时间只有一个活跃的线程能够访问数据，
       能够从设计上天然地避免线程竞争和数据冲突的问题；

# 什么时候使用 channel，什么时候使用 sync？

    参考：
        https://zhuanlan.zhihu.com/p/213712219

<p align='center'>
    <img src='./images/Go Primitive vs Channel.jpg'>
</p>

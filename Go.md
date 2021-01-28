#### slice 的底层实现

    切片本身并不是动态数组或者数组指针。它内部实现的数据结构通过指针引用底层数组，设定相关属性将数据读写操作限定在指定的区域内。
    ```
    type slice struct {
        array unsafe.Pointer
        len   int
        cap   int
    }
    ```
    切片的结构体由3部分构成，Pointer 是指向一个数组的指针，len 代表当前切片的长度，cap 是当前切片的容量。cap 总是大于等于 len 的。

#### 切片扩容策略

    1. 首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）
    2. 否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap）
    3. 否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的 1/4，
       即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
    4. 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）

    注意：扩容扩大的容量都是针对原来的容量而言的，而不是针对原来数组的长度而言的。

    切片扩容后是新数组 or 老数组？
        1. 由于原数组还有容量可以扩容，所以执行 append() 操作以后，会在原数组上直接操作，
           所以这种情况下，扩容以后的数组还是指向原来的数组。
        2. 如果原来数组的容量已经达到了最大值，再想扩容，Go 默认会先开一片内存区域，把原来的值拷贝过来，然后再执行 append() 操作。
           这种情况丝毫不影响原数组。

#### Map 使用什么方法解决 hash 冲突？

    拉链法

#### Map 扩容的过程

    参考：https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/

    哈希在存储元素过多时会触发扩容操作，每次都会将桶的数量翻倍，扩容过程不是原子的，而是通过 runtime.growWork 增量触发的，
    在扩容期间访问哈希表时会使用旧桶，向哈希表写入数据时会触发旧桶元素的分流。

#### 使用运算符"+"连接字符串会有性能问题吗？

    使用运算符"+"连接字符串每次都会产生新的字符串，涉及到拷贝，
    并且会产生很多临时的无用的字符串，给 gc 带来额外的负担，所以性能比较差。

#### Go 在函数参数传递的时候是值传递还是引用传递？

    无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝。

    整型和数组类型在调用函数时会对内容进行拷贝，不影响原值。如果当前数组的大小非常的大，这种传值的方式会对性能造成比较大的影响。
    传递结构体时，会拷贝结构体中的全部内容。
    传递结构体指针时，会拷贝结构体指针。

#### 反射

    Go 语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法，但是在编译时并不知道这些变量的具体类型，这称为反射机制。

    三大法则：
        1. 从 interface{} 变量可以反射出反射对象；
        2. 从反射对象可以获取 interface{} 变量；
        3. 要修改反射对象，其值必须可设置。

#### 下面的 for range 代码是死循环么？

    ```
    func main() {
        v := []int{1, 2, 3}
        for i := range v {
            v = append(v, i)
        }
    }
    ```

    不会，在遍历之前会对切片复制，遍历的次数不会随着切片的变化而变化

#### 对大数组 for range 遍历有什么问题？

    ```
     //假设值都为1，这里只赋值3个
    var arr = [102400]int{1, 1, 1} 
    for i, n := range arr {
        //just ignore i and n for simplify the example
        _ = i 
        _ = n 
    }
    ```

    遍历前会对数组进行拷贝，浪费内存并且影响性能，可以对数组取地址遍历 for i, n := range &arr，或者对数组做切片引用 for i, n := range arr[:]

#### for range 对遍历的元素取地址加到另外一个切片后有什么问题？

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

    在 for range 中，变量 v 每次迭代的值都是赋值给 v，该变量的内存地址始终未变，这样把它的地址追加到新的切片中，该切片保存的都是同一个地址。
    可以在 newArr = append(newArr, &v) 前面加上 v := v，可以通过下标取原切片的值

#### 迭代修改变量问题

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














































































#### make 和 new 的区别

#### channel 实现原理

#### channel 关闭后读写

#### sync 包

#### mutex

#### Go 的垃圾回收机制

    todo

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

#### Goroutine 原理

    goroutine 的实现方式目前主要用 GPM 模型来描述。
    GPM 其中 G 是指 Goroutine,P是指Process(逻辑调度器）,M是指Machine
    todo

#### Goroutine 和 Python 里面的协程有什么区别？

    todo










































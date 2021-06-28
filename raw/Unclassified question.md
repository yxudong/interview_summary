#### 1 到 n 中减少了一个数，顺序被打乱，找出缺失的数？

    1. 用 1+2+...+n 减去当前输入数据的总和。（容易溢出）
    2. 对输入数据进行 Hash（可以 bitmap），然后从头到尾遍历一次。（容易溢出）
    3. 用 1^2^...^n 的结果在逐个异或当前输入数据。

#### 给定一个奇数位升序，偶数位降序的链表，将其重新排序。

    输入: 1->8->3->6->5->4->7->2->NULL
    输出: 1->2->3->4->5->6->7->8->NULL

    1. 按奇偶位置拆分链表，得 1->3->5->7->NULL 和 8->6->4->2->NULL
    2. 反转偶链表，得 1->3->5->7->NULL 和 2->4->6->8->NULL
    3. 合并两个有序链表，得 1->2->3->4->5->6->7->8->NULL

#### 天平称重问题

    问题：
        用天平（只能比较，不能称重）从一堆小球中找出其中唯一一个较轻的，使用 x 次天平，
        最多可以从 y 个小球中找出较轻的那个，求 y 与 x 的关系式？

    将小球拆分为 3 份，在相同的比较次数下，可提高小球的比较数量，因此，其 y 与 x 的关系式为：y = 3^x

#### 抢 30 问题

    甲乙二人轮流报数。从 1 起，每人每次可报一个数或连续报两个数。谁能报得 20 谁就获胜。
    先和同学玩一玩这个游戏。如果由你先报数，你能保证获胜吗？

        可以从 20 往前想，如果想获胜，自己不要报 19 和 18。
        因为报 19，对方报 20 这一个数就获胜了；
        报 18，对方连续报两个数 19、20 就获胜了。这样，要想获胜（抢到 20）必须抢到 17。
        同理，要想抢到 17，就要争取抢到 14；
        要想抢到 14，就要争取抢到 11；
        要想抢到 11，就要争取抢到 8；
        要想抢到 8，就要争取抢到 5；
        要想抢到 5，就要争取抢到 2；
        因此，先抢到 2。
        对方报 3，自己报 4、5；对方报 3、4，自己报 5。这样就又抢到了 5。
        依次方法继续下去，就一定会获胜了。

    按照上面的报数方法，如果先报 20 的一方失败，怎样保证获胜？

        让 20 就要抢 19，并且依次抢 16、13、10、7、4、1。
        因此，要先报 1，再根据对方报数情况依次抢 4、7、10、13、16、19，这样就把 20 让给了对方。

#### 限流算法

    参考：
        1. [一文搞懂高频面试题之限流算法，从算法原理到实现，再到对比分析](https://juejin.cn/post/6870396751178629127#heading-12)
        2. [图解+代码|常见限流算法以及限流在单机分布式场景下的思考](https://segmentfault.com/a/1190000023552181)
        3. [分布式服务限流实战，已经为你排好坑了](https://www.infoq.cn/article/qg2tx8fyw5vt-f3hh673)

    1. 计数器固定窗口算法
        计数器固定窗口算法是最基础也是最简单的一种限流算法。
        原理就是对一段固定时间窗口内的请求进行计数，如果请求数超过了阈值，则舍弃该请求；如果没有达到设定的阈值，则接受该请求，且计数加 1。
        当时间窗口结束时，重置计数器为 0。

        优点：实现简单，容易理解。
        缺点：
            1.1 流量曲线可能不够平滑，有“突刺现象”，一段时间内（不超过时间窗口）系统服务不可用。
                比如窗口大小为 1s，限流大小为 100，然后恰好在某个窗口的第 1ms 来了 100 个请求，
                然后第 2ms-999ms 的请求就都会被拒绝，这段时间用户会感觉系统服务不可用。
            1.2 窗口切换时可能会产生两倍于阈值流量的请求。
                比如窗口大小为 1s，限流大小为 100，然后恰好在某个窗口的第 999ms 来了 100 个请求，窗口前期没有请求，所以这 100 个请求都会通过。
                再恰好，下一个窗口的第 1ms 有来了 100 个请求，也全部通过了，那也就是在 2ms 之内通过了 200 个请求，
                而我们设定的阈值是 100，通过的请求达到了阈值的两倍。
    2. 计数器滑动窗口算法
        计数器滑动窗口算法是计数器固定窗口算法的改进，解决了固定窗口切换时可能会产生两倍于阈值流量请求的缺点。
        滑动窗口算法在固定窗口的基础上，将一个计时窗口分成了若干个小窗口，然后每个小窗口维护一个独立的计数器。
        当请求的时间大于当前窗口的最大时间时，则将计时窗口向前平移一个小窗口。
        平移时，将第一个小窗口的数据丢弃，然后将第二个小窗口设置为第一个小窗口，同时在最后面新增一个小窗口，将新的请求放在新增的小窗口中。
        同时要保证整个窗口中所有小窗口的请求数目之后不能超过设定的阈值。

        相对于固定窗口，滑动窗口除了需要引入计数器之外还需要记录时间窗口内每个请求到达的时间点，因此对内存的占用会比较多。
        但是滑动窗口和固定窗口都无法解决短时间之内集中流量的突击。

    3. 漏斗算法

<p align='center'>
    <img src='./images/Unclassified question/漏桶算法.png'>
</p>

        漏桶算法概念如下：
            - 将每个请求视作“水滴”放入“漏桶”进行存储；
            - “漏桶”以固定速率向外“漏”出请求来执行如果“漏桶”空了则停止“漏水”；
            - 如果“漏桶”满了则多余的“水滴”会被直接丢弃。
        漏桶算法多使用队列实现，服务的请求会存到队列中，服务的提供方则按照固定的速率从队列中取出请求并执行，过多的请求则放在队列中排队或直接拒绝。

        漏桶算法的缺陷也很明显，当短时间内有大量的突发请求时，即便此时服务器没有任何负载，每个请求也都得在队列中等待一段时间才能被响应。

    4. 令牌桶算法

<p align='center'>
    <img src='./images/Unclassified question/令牌桶算法.webp'>
</p>

        令牌桶算法概念如下：
            - 令牌以固定速率生成；
            - 生成的令牌放入令牌桶中存放，如果令牌桶满了则多余的令牌会直接丢弃，
              当请求到达时，会尝试从令牌桶中取令牌，取到了令牌的请求可以执行；
            - 如果桶空了，那么尝试取令牌的请求会被直接丢弃。

        令牌桶算法既能够将所有的请求平均分布到时间区间内，又能接受服务器能够承受范围内的突发请求，因此是目前使用较为广泛的一种限流算法。

    分布式限流可以使用 Redis+Lua 脚本的方式实现上述算法。

#### 快速排序的时间复杂度怎么计算的？

    todo

#### 快速排序优化？

    三数中值
    todo

#### 一致性哈希

    todo

#### CAS

    todo

#### 从无限的字符流中, 随机选出 10 个字符

    todo

#### 分布式事务

    todo

#### 跳表的实现方法

    todo

#### raft 算法

    todo

#### 负载均衡的相关算法

    todo





































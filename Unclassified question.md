<a name="index">**目录：**</a><br>
&emsp;&emsp;<a href="#0">1 到 n 中减少了一个数，顺序被打乱，找出缺失的数</a><br>
&emsp;&emsp;<a href="#1">给定一个奇数位升序，偶数位降序的链表，将其重新排序。</a><br>
&emsp;&emsp;<a href="#2">天平称重问题</a><br>
&emsp;&emsp;<a href="#3">抢 30 问题</a><br>
&emsp;&emsp;<a href="#4">服务限流</a><br>
&emsp;&emsp;<a href="#5">服务熔断</a><br>
&emsp;&emsp;<a href="#6">快速排序优化？</a><br>
&emsp;&emsp;<a href="#7">从无限的字符流中, 随机选出 10 个字符</a><br>
&emsp;&emsp;<a href="#8">分布式共识算法（Distributed Consensus Algorithm）</a><br>
&emsp;&emsp;<a href="#9">分布式事务</a><br>
&emsp;&emsp;<a href="#10">跳表的实现方法</a><br>
&emsp;&emsp;<a href="#11">一致性哈希</a><br>
&emsp;&emsp;<a href="#12">CAS</a><br>
&emsp;&emsp;<a href="#13">负载均衡的相关算法</a><br>
&emsp;&emsp;<a href="#14">avl tree b树与b+树，红黑树的区别和应用场景？</a><br>
&emsp;&emsp;<a href="#15">top k 问题</a><br>
&emsp;&emsp;<a href="#16">红黑树平衡性，复杂度</a><br>
&emsp;&emsp;<a href="#17">朋友圈设计</a><br>
&emsp;&emsp;<a href="#18">日志写入系统</a><br>
&emsp;&emsp;<a href="#19">海量评论系统</a><br>
&emsp;&emsp;<a href="#20">多活系统</a><br>
# <a name="0">1 到 n 中减少了一个数，顺序被打乱，找出缺失的数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    1. 用 1+2+...+n 减去当前输入数据的总和。（容易溢出）
    2. 对输入数据进行 Hash（可以 bitmap），然后从头到尾遍历一次。（容易溢出）
    3. 用 1^2^...^n 的结果再逐个异或当前输入数据。

# <a name="1">给定一个奇数位升序，偶数位降序的链表，将其重新排序。</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    输入: 1->8->3->6->5->4->7->2->NULL
    输出: 1->2->3->4->5->6->7->8->NULL

    1. 按奇偶位置拆分链表，得 1->3->5->7->NULL 和 8->6->4->2->NULL
    2. 反转偶链表，得 1->3->5->7->NULL 和 2->4->6->8->NULL
    3. 合并两个有序链表，得 1->2->3->4->5->6->7->8->NULL

# <a name="2">天平称重问题</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    问题：
        用天平（只能比较，不能称重）从一堆小球中找出其中唯一一个较轻的，使用 x 次天平，
        最多可以从 y 个小球中找出较轻的那个，求 y 与 x 的关系式？

    将小球拆分为 3 份，在相同的比较次数下，可提高小球的比较数量，因此，其 y 与 x 的关系式为：y = 3^x

# <a name="3">抢 30 问题</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="4">服务限流</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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
                比如窗口大小为 1s，限流大小为 100，然后恰好在某个窗口的第 999ms 来了 100 个请求，
                窗口前期没有请求，所以这 100 个请求都会通过。
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
        漏桶算法多使用队列实现，服务的请求会存到队列中，服务的提供方则按照固定的速率从队列中取出请求并执行，
        过多的请求则放在队列中排队或直接拒绝。

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

# <a name="5">服务熔断</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. [服务容错模式](https://tech.meituan.com/2016/11/11/service-fault-tolerant-pattern.html)

    熔断器模式可以防止系统不断地尝试执行可能会失败的调用，使得系统继续执行而不用等待修正错误，或者浪费 CPU 时间去等到长时间的超时产生。
    熔断器模式也可以使系统能够检测错误是否已经修正，如果已经修正，系统会再次尝试调用操作。下图是个使用熔断器模式的调用流程：

<p align='center'>
    <img src='./images/Unclassified question/熔断器模式 1.png'>
</p>

    可以从图中看出，当超时出现的次数达到一定条件后，熔断器会触发打开状态，客户端的下次调用将直接返回，不用等待超时产生。

    在熔断器内部，往往有以下几种状态：

<p align='center'>
    <img src='./images/Unclassified question/熔断器模式 2.png'>
</p>

    1. 闭合（closed）状态：
        该状态下能够对目标服务或方法进行正常的调用。
        熔断器类维护了一个时间窗口内调用失败的次数，如果某次调用失败，则失败次数加 1。
        如果最近失败次数超过了在给定的时间窗口内允许失败的阈值（可以是数量也可以是比例），则熔断器类切换到断开（Open）状态。
        此时熔断器设置了一个计时器，当时钟超过了该时间，则切换到半断开（Half-Open）状态，
        该睡眠时间的设定是给了系统一次机会来修正导致调用失败的错误。
    2. 断开（Open）状态：
        在该状态下，对目标服务或方法的请求会立即返回错误响应，如果设置了 fallback 方法，则会进入 fallback 的流程。
    3. 半断开（Half-Open）状态：
        允许对目标服务或方法的一定数量的请求可以去调用服务。
        如果这些请求对服务的调用成功，那么可以认为之前导致调用失败的错误已经修正，此时熔断器切换到闭合状态（并且将错误计数器重置）；
        如果这一定数量的请求有调用失败的情况，则认为导致之前调用失败的问题仍然存在，熔断器切回到断开方式，
        然后开始重置计时器来给系统一定的时间来修正错误。半断开状态能够有效防止正在恢复中的服务被突然而来的大量请求再次拖垮。

# <a name="6">快速排序优化？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    三数（最开始，最末尾，中间）取中值，为了使基准值更加随机，避免待排序数组已经基本有序，时间复杂度退化

# <a name="7">从无限的字符流中, 随机选出 10 个字符</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. [用Python写算法 | 蓄水池算法实现随机抽样](https://juejin.cn/post/6844903651706667022)
        2. [Reservoir Sampling - 蓄水池抽样](https://www.cnblogs.com/HappyAngel/archive/2011/02/07/1949762.html)
        3. [蓄水池抽样算法（Reservoir Sampling）](https://www.jianshu.com/p/7a9ea6ece2af)

    蓄水池采样算法:
    算法思路大致如下：

        先把前 k 个数放入蓄水池，
        对第 i 个数，以 k/i 概率决定是否要把它换入蓄水池，换入时随机的选取一个作为替换项，这样一直做下去，
        对于任意的样本空间 n，对每个数的选取概率都为 k/n。也就是说对每个数选取概率相等。

<p align='center'>
    <img src='./images/Unclassified question/蓄水池采样算法证明.png'>
</p>

    分布式蓄水池抽样：
        1. 假设有 K 台机器，将大数据集分成 K 个数据流，每台机器使用单机版蓄水池抽样处理一个数据流，抽样 m 个数据，
           并最后记录处理的数据量为 N1，N2，...，Nk，...，NK（假设 m < Nk）。N1+N2+...+NK=N。
        2. 取 [1, N] 一个随机数 d，
           若 d<N1，则在第一台机器的蓄水池中等概率不放回地（1/m）选取一个数据；
           若 N1<=d<(N1+N2)，则在第二台机器的蓄水池中等概率不放回地选取一个数据；
           依此类推，重复 m 次，则最终从 N 大数据集中选出 m 个数据。

        m/N 的概率验证如下：
            1. 在第 k 台机器中的蓄水池，某个数据被选取的概率为 m/Nk。
            2. 从第 k 台机器的蓄水池中选取一个数据放进最终蓄水池的概率为 Nk/N。
            3. 第 k 台机器蓄水池的一个数据被选中的概率为 1/m。（不放回选取时等概率的）
            4. 重复 m 次选取，则每个数据被选中的概率为 m*(m/Nk*Nk/N*1/m)=m/N。

# <a name="8">分布式共识算法（Distributed Consensus Algorithm）</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    http://icyfenix.cn/distribution/consensus/gossip.html
    https://tech.youzan.com/cap-coherence-protocol-and-application-analysis/
    https://juejin.cn/post/6977184839447363592#heading-8
    https://catkang.github.io/2017/06/30/raft-subproblem.html
    https://my.oschina.net/pingpangkuangmo/blog/776714
    https://juejin.cn/post/6907151199141625870#heading-30
    https://mp.weixin.qq.com/s?__biz=MzAwMjI0ODk0NA==&mid=2451950743&idx=1&sn=df1c600f636c8d9b119f534750c007eb&chksm=8d1c3508ba6bbc1e6e4def2ea4c25d9c5e69013d463af31f6bc78cacbc3735ccea455842303d&scene=21#wechat_redirect

# <a name="9">分布式事务</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="10">跳表的实现方法</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="11">一致性哈希</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    https://juejin.cn/post/6844903750860013576
    https://www.jianshu.com/p/4163916a2a8a

# <a name="12">CAS</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="13">负载均衡的相关算法</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="14">avl tree b树与b+树，红黑树的区别和应用场景？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    红黑树与avl树的区别，为什么有这些区别？

# <a name="15">top k 问题</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="16">红黑树平衡性，复杂度</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="17">朋友圈设计</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    https://cloud.tencent.com/developer/article/1082731
    扩散写

# <a name="18">日志写入系统</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    https://cloud.tencent.com/developer/article/1328501
    https://juejin.cn/post/6922811488784613389

# <a name="19">海量评论系统</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    像知乎这样的评论回复模式，数据库应该怎么设计比较好，考虑性能、可扩展性等？ - Syter的回答 - 知乎
    https://www.zhihu.com/question/38959595/answer/79084228

    https://github.com/aylei/interview#%E4%B8%89%E9%9D%A2-2

# <a name="20">多活系统</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

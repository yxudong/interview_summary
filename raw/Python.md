# Python 中的整数

    参考：
        1. [《深度剖析CPython解释器》5. 解密Python中的整数在底层是如何实现的，以及为什么Python中大整数的运算不会溢出](https://www.cnblogs.com/traditional/p/13437243.html)

    Python 中整数对应的值在底层是使用数组来存储的，所以理论上永远不会溢出。

# Dict 底层实现原理

    参考：
        1. [《深度剖析CPython解释器》9. 解密Python中字典和集合的底层实现，深度分析哈希表](https://www.cnblogs.com/traditional/p/13503114.html)
        2. [Python 字典](https://flaggo.github.io/python3-source-code-analysis/objects/dict-object/)

    Python 3.6 之前维护一张 hash 表（键值对数组），字典是无序的；
    Python 3.7 之后同时哈希索引数组 和 键值对数组，字典是有序的。
    容量最小为 8。

<p align='center'>
    <img src='./images/Python-Dict 底层结构.png'>
</p>

    split-table dictionaries（分离表） vs combined-table dictionaries（结合表）：
        split-table dictionaries：
            当被创建的字典是用来保存 object 的__dict__属性时，该字典才会创建为一个 split-table，
            它们的键表都被缓存在类型属性中，并且允许所有该类型的实例都可以共享该 keys。
        combined-table dictionaries：
            直接通过 dict 內建函数与{}生成的字典，模块和大部分其他字典都会创建为 combined-table 字典。

    扩容规则：
        entry 个数达到容量的三分之二，会调整容量。
        大小是已用 entry 的 3 倍。

    如何处理 hash 冲突？
        使用开放地址法，使用探测函数进行探测
        Python 对此进行了优化，探测函数参考对象哈希值，生成不同的探测序列，进一步降低哈希冲突的可能性。

# Python List 底层实现原理

    参考：
        1. [《深度剖析CPython解释器》8. 解密Python中列表的底层实现，通过源码分析列表支持的相关操作](https://www.cnblogs.com/traditional/p/13461463.html)

    ```
    typedef struct {
        PyObject_VAR_HEAD
        PyObject **ob_item;
        Py_ssize_t allocated;
    } PyListObject;
    ```

    PyObject_VAR_HEAD: 变长对象的公共头部信息
    ob_item：一个二级指针，指向一个PyObject *类型的指针数组，这个指针数组保存的便是对象的指针，
             而操作底层数组都是通过ob_item来进行操作的。
    allocated：容量, 我们知道列表底层是使用了C的数组, 而底层数组的长度就是列表的容量

<p align='center'>
    <img src='./images/Python-List 底层结构.png'>
</p>

# Python 中的小整数池

    为了避免大量对象的创建和销毁，Python 将那些使用频率高的整数预先创建好，
    这些预先创建好的整数会放在一个静态数组里面，称为小整数对象池。如果需要使用的话会直接拿来用，而不用重新创建。

# def func(a,b=[]) 这种写法有什么坑？

    ```
    def func(a,b = []):
        b.append(1)
        print(a,b)

    func(a=2)
    func(2)
    func(2)
    ```
    '''
    2 [1]
    2 [1, 1]
    2 [1, 1, 1]
    函数的默认参数是一个 list 当第一次执行的时候实例化了一个 list
    第二次执行还是用第一次执行的时候实例化的地址存储
    所以三次执行的结果就是 [1, 1, 1] 想每次执行只输出 [1]，默认参数应该设置为 None
    '''

# is 和 == 的区别

    is 比较的是两个实例对象是不是完全相同，它们是不是同一个对象，占用的内存地址是否相同，相当于调用 id() 方法。
    == 比较的是两个对象的内容是否相等，即内存地址可以不一样，内容一样就可以了，默认会调用对象的 __eq__()方法。

# Python 递归的最大层数？

    默认是 1000
    可以通过 sys.setrecursionlimit() 进行设置

# a = [1, 2, 3, 4, 5]，b = a 和 b = a[:]，有区别么？

    前者传递引用，后者是拷贝

# 什么是面向对象的mro
# 面向对象深度优先和广度优先是什么？
# 经典类和新式类















# 多个装饰器的执行顺序
# 静态方法和类方法，普通方法
# Python 的元类
# __init__, __new__ 区别
# 怎么理解 __call__
# Python 垃圾回收机制
# order_dict 怎么实现，3.8 开始字典有序
# yield 关键字作用
# await 和 async 等关键字，版本，作用
# tornado 实现原理
# 可变对象和不可变对象
# GIL
# 怎么实现单例
# super 作用
# 列表推导式，字典推导式，生成器推导式
# list() 和 [] 哪个效率更快？
# property 作用
# __dict__ 内容
# slot 作用
# 介绍一下Python 锁？
# 什么是python的生成器?
# 装饰器的写法以及应用场景。
# 什么是反射？以及应用场景？
# *arg和**kwarg作用
# xrange和range的区别
# Python 的 sort 函数实现原理
# Python 装饰器的原理
# 一行代码写出乘法表
    print("\n".join([" ".join(["%s*%s=%-2s"%(j,i,j*i) for j in range(1,i+1)]) for i in range(1,10)]))
# 代码题:实现一个生产者消费者
# Python 线程安全
    https://segmentfault.com/a/1190000022644337














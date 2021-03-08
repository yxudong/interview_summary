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

# def func(a, b=[]) 这种写法有什么坑？

    ```
    def func(a, b=[]):
        b.append(1)
        print(a, b)

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

# Python 中的 MRO

    参考：
        1. [Python的方法解析顺序(MRO)](https://hanjianwei.com/2013/07/25/python-mro/)
        2. [Python多重继承排序原理（MRO算法解析，拓扑排序，C3算法）](https://www.pianshen.com/article/6817106786/)

    在方法调用时就需要对当前类和基类进行搜索以确定方法所在的位置。
    而搜索的顺序就是所谓的「方法解析顺序」（Method Resolution Order，或 MRO）。

    Python 有三种不同的 MRO：
        1. 经典类（classic class）的深度遍历。
        2. Python 2.2 的新式类（new-style class）预计算。
        3. Python 2.3 的新式类的 C3 算法。它也是 Python 3 唯一支持的方式。

    注意：
        1. 新式类中的算法（指上面的第 3 种情况）并不完全是广度优先算法，而是 C3 算法。
        2. C3 算法不完全和拓扑排序相等，而是在拓扑排序的基础上，考虑了基类的出现顺序。

# 多个装饰器的执行顺序

    参考：
        1. [理解 Python 装饰器看这一篇就够了](https://foofish.net/python-decorator.html)

    ```
    @a
    @b
    @c
    def f ():
        pass

    # 它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于
    # f = a(b(c(f)))
    ```

# 类装饰器

    使用类装饰器主要依靠类的__call__方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

    ```
    class Foo(object):
        def __init__(self, func):
            self._func = func

        def __call__(self):
            print ('class decorator runing')
            self._func()
            print ('class decorator ending')

    @Foo
    def bar():
        print ('bar')

    bar()
    ```

# 静态方法和类方法，普通方法

    参考：
        1. [Python类的静态方法和类方法区别](https://zhuanlan.zhihu.com/p/21101992)'

    实例方法只能被实例对象调用（Python3 中，如果类调用实例方法，需要显示的传self, 也就是实例对象自己)，
    静态方法(由@staticmethod装饰的方法)、类方法(由@classmethod装饰的方法)，可以被类或类的实例对象调用。

    实例方法，第一个参数必须要默认传实例对象，一般习惯用 self。
    静态方法，参数没有要求。
    类方法，第一个参数必须要默认传类，一般习惯用 cls。

    类方法可以用在模拟定义多个构造函数的情况。由于 Python 类中只能有一个初始化方法，不能按照不同的情况初始化类。
    ```
    class Book(object):

        def __init__(self, title):
            self.title = title

        @classmethod
        def create(cls, title):
            book = cls(title=title)
            return book

    book1 = Book("python")
    book2 = Book.create("python and django")
    print(book1.title)
    print(book2.title)
    ```

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














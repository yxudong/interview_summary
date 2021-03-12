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

    参考：
        1. [Python进阶——详解元类，metaclass的原理和用法](https://zhuanlan.zhihu.com/p/149126959)
        2. [使用元类](https://www.liaoxuefeng.com/wiki/1016959663602400/1017592449371072#0)
        3. [关于Python中的type, object, class, 和 instance](https://zhuanlan.zhihu.com/p/353391573)
        4. [陌生的 metaclass](https://wiki.jikexueyuan.com/project/explore-python/Class/metaclass.html)

    在 Python 中，一切皆对象。字符串，列表，字典，函数是对象，类也是一个对象，因此可以：
        - 把类赋值给一个变量
        - 把类作为函数参数进行传递
        - 把类作为函数的返回值
        - 在运行时动态地创建类

    动态语言和静态语言最大的不同，就是函数和类的定义，不是编译时定义的，而是运行时动态创建的。
    type 除了可以返回对象的类型，它还可以被用来动态地创建类（对象）。
    使用 type 来创建类（对象）的方式如下：
        type(类名, 父类的元组（针对继承的情况，可以为空），包含属性和方法的字典（名称和值）)

    ```
    class Foo(object):
        pass
    # 不使用 class 关键字来定义，而使用 type，如下：

    Foo = type('Foo', (object, ), {})    # 使用 type 创建了一个类对象
    ```

    什么是元类：
        元类（metaclass）是用来创建类（对象）的可调用对象。
        这里的可调用对象可以是函数或者类等。但一般情况下，使用类作为元类。
        对于实例对象、类和元类，可以用下面的图来描述：
            类是实例对象的模板，元类是类的模板

            +----------+             +----------+             +----------+
            |          |             |          |             |          |
            |          | instance of |          | instance of |          |
            | instance +------------>+  class   +------------>+ metaclass|
            |          |             |          |             |          |
            |          |             |          |             |          |
            +----------+             +----------+             +----------+

        前面使用了 type 来创建类（对象），事实上，type 就是一个元类。
        元类的主要目的是为了控制类的创建行为。

        当定义 class Bar(Foo) 时，Python 会首先在当前类，即 Bar 中寻找 __metaclass__，
        如果没有找到，就会在父类 Foo 中寻找 __metaclass__，如果找不到，就继续在 Foo 的父类寻找，
        如此继续下去，如果在任何父类都找不到 __metaclass__，就会到模块层次中寻找，
        如果还是找不到，就会用 type 来创建这个类。

    type, object, class, 和 instance 关系：

<p align='center'>
    <img src='./images/Python-type, object, class 和 instance（1）.png'>
</p>
        object 在这张图中的角色很重要，它既是所有类的基类（base class）（所有类都继承它），
        也是 type 类（注意 type 也是类）的实例。
        type 的实例怎么会是一个类呢？答案是 type 本身的类是一种“类的类”即“元类”（metaclass）。
        元类 type 规定了 object 这个所有类的基类应该长什么样子，
        按照元类 type 的模子产生的实例 object 自然也就成了一个类。
        但是为什么又说 type 继承 object 呢？因为 type 本身也是一个类啊，甚至可以认为 object 先于 type，
        因为 'help(object)' 的执行结果第一行是 'class object'，object 就不继承任何类了；
        而 'help(type)' 的执行结果第一行是 'class type(object)'，说明 type 继承 object，
        它虽然是类的类，但“类”这个身份也恰恰让它不得不符合类的基本形式。上面这张图还可以画成这种形式：

<p align='center'>
    <img src='./images/Python-type, object, class 和 instance（2）.png'>
</p>

    用途：
        1. ORM 框架
        2. 限制某些类不能直接实例化，必须要通过特定的类返回实例（通过 __call__ 方法）。

# __call__ 方法

    object() 相当于 object.__call__()
    __call__() 的作用是使实例能够像函数一样被调用，
    同时不影响实例本身的生命周期（__call__()不影响一个实例的构造和析构）。

    ```
    class Foo:
        def __call__(self, a, b, c):
            # ...

    x = Foo()
    x(1, 2, 3) # __call__
    ```

# OrderedDict 怎么实现

    在继承了自身的 Dict 之后，类中还声明了一个循环双向链表 self.__root 作为自己的属性，
    用另一个 Dict 结构 self.__map 来存储 Key 到链表节点的对应关系，
    可以类比 LRU 缓存的实现方法（Leetcode）。

    Python 3.7 中原生 Dict 与 OrderedDict 的区别：
        1. OrderedDict 支持对 Key 的顺序有关的操作，
           比如说把某个 Key 挪到头部或者尾部，
           逆序给出 Key 的序列（这个特性在 Python 3.8 中加到了原生 Dict 中）等；
        2. OrderedDict 比较的时候会把 Key 的插入顺序考虑进去，而 Dict 不会

# Python 线程安全

    参考：
        1. [通俗易懂：说说 Python 里的线程安全、原子操作](https://segmentfault.com/a/1190000022644337)
        2. https://docs.python.org/3.7/faq/library.html#what-kinds-of-global-value-mutation-are-thread-safe

    只有原子操作是线程安全的，比如
        L.append(x)
        L1.extend(L2)
        x = L[i]
        x = L.pop()
        L1[i:j] = L2
        L.sort()
        x = y
        x.field = y
        D[x] = y
        D1.update(D2)
        D.keys()
    而下面这些就不是原子操作
        i = i+1
        L.append(L[-1])
        L[i] = L[j]
        D[x] = D[x] + 1

    实现人工原子操作可以通过加锁的方式。

# Python 的 sort 函数实现原理

    参考：
        1. [python sort函数内部实现原理](https://www.cnblogs.com/clement-jiao/p/9243066.html)

    内部实现机制为：Timesort
    最坏时间复杂度为：O（n log n）
    空间复杂度为：O（n）
    算法原理：
       Timsort 是结合了合并排序（merge sort）和插入排序（insertion sort）而得出的排序算法，它在现实中有很好的效率。
       该算法找到数据中已经排好序的块-分区，每一个分区叫一个 run，然后按规则合并这些 run。

# xrange() 和 range() 的区别

    Python2 中 xrange() 返回生成器，range() 返回 list，
    Python3 中没有 xrange()，只有 range()，并且返回生成器。

# Python 中的 GIL

    参考：
        1. [Python GIL(Global Interpreter Lock)](https://www.cnblogs.com/hwlong/p/9002779.html)
        2. [谈谈python的GIL、多线程、多进程](https://zhuanlan.zhihu.com/p/20953544)

    首先需要明确的一点是 GIL 并不是 Python 的特性，它是在实现 Python 解析器（CPython）时所引入的一个概念。
    因为 CPython 是大部分环境下默认的 Python 执行环境。所以在很多人的概念里 CPython 就是 Python。

    在 Python 多线程下，每个线程的执行方式：
        1. 获取 GIL
        2. 执行代码直到 sleep 或者是 Python 虚拟机将其挂起。
        3. 释放 GIL
    可见，某个线程想要执行，必须先拿到 GIL，我们可以把 GIL 看作是“通行证”，并且在一个 Python 进程中，GIL只有一个。
    拿不到通行证的线程，就不允许进入 CPU 执行。

    为什么会有 GIL：
        实现简单，保证内存数据线程安全

    怎么解决：
        对于 CPU 密集型代码，使用多进程，或者改为 C 实现
        对于 IO 密集型代码，使用多线程

# Python 中的 __dict__ 和 dir()

    参考：
        1. [what's the biggest difference between dir and __dict__ in python](https://stackoverflow.com/questions/14361256/whats-the-biggest-difference-between-dir-and-dict-in-python)

    dir() 不仅查找对象的 __dict__（有时甚至不存在），它还会根据对象的继承关系提供所有可用属性。
    __dict__ 只是获取实例的“本地”属性集，并不包含该实例的所有可用属性。

# list() 和 [] 哪个效率更快？

    创建空列表时，[] 要比 list() 快。

    对于 []，它是 Python 中的一组字面量（literal），像数字之类的字面量一样，表示确切的固定值。
    也就是说，Python 在解析到它时，就知道它要表示一个列表，因此会直接调用解释器中构建列表的方法来创建列表。

    对于 list()，“list”只是一个普通的名称，并不是字面量，也就是说解释器一开始并不认识它。
    因此，解释器的第一步是要找到这个名称（对应 LOAD_NAME）。它会按照一定的顺序，
    在各个作用域中逐一查找（局部作用域--全局作用域--内置作用域），直到找到为止，找不到则会抛出NameError。
    第二步是把这个名称当作可调用对象来调用，即把它当成一个函数进行调用（对应 CALL_FUNCTION）。
    因此，list() 在创建列表时，需要经过名称查找与函数调用两个步骤，才能真正开始创建列表。

    因为 list() 涉及的执行步骤更多，因此它比 [] 要慢一些。

# 列表推导式，字典推导式，生成器推导式

    列表推导式：
        ```
        # 使用 [] 生成 list
        a = [i for i in range(10)]
        # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

        # 使用 () 生成 generator
        a = (i for i in range(10))
        # <generator object <genexpr> at 0x00000281C61DA840>
        ```

    字典推导式：
        ```
        a = {'a': 1, 'b': 2, 'c': 3}
        b = {v: k for k, v in a.items()}
        # {1: 'a', 2: 'b', 3: 'c'}
        ```

    集合推导式：
        ```
        a = {x**2 for x in [1, 1, 2]}
        # {1, 4}
        ```

# Python 装饰器

    参考：
        1. [理解 Python 装饰器看这一篇就够了](https://foofish.net/python-decorator.html)
        2. [一文看懂Python系列之装饰器(decorator)(工作面试必读)](https://zhuanlan.zhihu.com/p/51158386)

    装饰器是利用闭包原理，区别是装饰器在闭包中传入的参数是函数，而不是变量。

    简单装饰器：
        ```
        def use_logging(func):
            def wrapper():
                logging.warn("%s is running" % func.__name__)
                return func()

            return wrapper

        @use_logging
        def foo():
            print("i am foo")

        # foo = use_logging(foo)
        foo()
        ```

    带参数的装饰器:
        ```
        def use_logging(level):
            def decorator(func):
                def wrapper(*args, **kwargs):
                    if level == "warn":
                        logging.warn("%s is running" % func.__name__)
                    elif level == "info":
                        logging.info("%s is running" % func.__name__)
                    return func(*args)
                return wrapper

            return decorator

        # @use_logging(level="warn") 等价于 @decorator
        @use_logging(level="warn")
        def foo(name='foo'):
            print("i am %s" % name)

        foo()
        ```

    多个装饰器的执行顺序：
        ```
        @a
        @b
        @c
        def f ():
            pass

        # 它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于
        # f = a(b(c(f)))
        ```

    类装饰器：
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
















# __prepare__() 方法
# __init__, __new__ 区别
# Python 垃圾回收机制
# yield 关键字作用
# await 和 async 等关键字，版本，作用
# tornado 实现原理
# 可变对象和不可变对象
# 怎么实现单例
# super 作用
# 生成器原理
# list() 和 [] 哪个效率更快？
# property 作用
# slot 作用
# 介绍一下Python 锁？
# 什么是python的生成器?
# 什么是反射？以及应用场景？
# *arg和**kwarg作用
# 一行代码写出乘法表
    print("\n".join([" ".join(["%s*%s=%-2s"%(j,i,j*i) for j in range(1,i+1)]) for i in range(1,10)]))
# 代码题:实现一个生产者消费者















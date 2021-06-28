<a name="index">**目录：**</a><br>
&emsp;&emsp;<a href="#0">Python 中的整数</a><br>
&emsp;&emsp;<a href="#1">Dict 底层实现原理</a><br>
&emsp;&emsp;<a href="#2">Python List 底层实现原理</a><br>
&emsp;&emsp;<a href="#3">Python 中的小整数池</a><br>
&emsp;&emsp;<a href="#4">def func(a, b=[]) 这种写法有什么坑？</a><br>
&emsp;&emsp;<a href="#5">is 和 == 的区别</a><br>
&emsp;&emsp;<a href="#6">Python 递归的最大层数？</a><br>
&emsp;&emsp;<a href="#7">a = [1, 2, 3, 4, 5]，b = a 和 b = a[:]，有区别么？</a><br>
&emsp;&emsp;<a href="#8">Python 中的 MRO</a><br>
&emsp;&emsp;<a href="#9">静态方法和类方法，普通方法</a><br>
&emsp;&emsp;<a href="#10">Python 的元类</a><br>
&emsp;&emsp;<a href="#11">  # 不使用 class 关键字来定义，而使用 type，如下：</a><br>
&emsp;&emsp;<a href="#12">__call__ 方法</a><br>
&emsp;&emsp;<a href="#13">          # ...</a><br>
&emsp;&emsp;<a href="#14">OrderedDict 怎么实现</a><br>
&emsp;&emsp;<a href="#15">Python 线程安全</a><br>
&emsp;&emsp;<a href="#16">Python 的 sort 函数实现原理</a><br>
&emsp;&emsp;<a href="#17">xrange() 和 range() 的区别</a><br>
&emsp;&emsp;<a href="#18">Python 中的 GIL</a><br>
&emsp;&emsp;<a href="#19">Python 中的 __dict__ 和 dir()</a><br>
&emsp;&emsp;<a href="#20">list() 和 [] 哪个效率更快？</a><br>
&emsp;&emsp;<a href="#21">列表推导式，字典推导式，集合推导式</a><br>
&emsp;&emsp;<a href="#22">      # 使用 [] 生成 list</a><br>
&emsp;&emsp;<a href="#23">      # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]</a><br>
&emsp;&emsp;<a href="#24">      # 使用 () 生成 generator</a><br>
&emsp;&emsp;<a href="#25">      # <generator object <genexpr> at 0x00000281C61DA840></a><br>
&emsp;&emsp;<a href="#26">      # {1: 'a', 2: 'b', 3: 'c'}</a><br>
&emsp;&emsp;<a href="#27">      # {1, 4}</a><br>
&emsp;&emsp;<a href="#28">Python 装饰器</a><br>
&emsp;&emsp;<a href="#29">      # foo = use_logging(foo)</a><br>
&emsp;&emsp;<a href="#30">      # @use_logging(level="warn") 等价于 @decorator</a><br>
&emsp;&emsp;<a href="#31">      # 它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于</a><br>
&emsp;&emsp;<a href="#32">      # f = a(b(c(f)))</a><br>
&emsp;&emsp;<a href="#33">一行代码写出乘法表</a><br>
&emsp;&emsp;<a href="#34">  # 1*1=1</a><br>
&emsp;&emsp;<a href="#35">  # 1*2=2 2*2=4</a><br>
&emsp;&emsp;<a href="#36">  # 1*3=3 2*3=6 3*3=9</a><br>
&emsp;&emsp;<a href="#37">  # 1*4=4 2*4=8 3*4=12 4*4=16</a><br>
&emsp;&emsp;<a href="#38">  # 1*5=5 2*5=10 3*5=15 4*5=20 5*5=25</a><br>
&emsp;&emsp;<a href="#39">  # 1*6=6 2*6=12 3*6=18 4*6=24 5*6=30 6*6=36</a><br>
&emsp;&emsp;<a href="#40">  # 1*7=7 2*7=14 3*7=21 4*7=28 5*7=35 6*7=42 7*7=49</a><br>
&emsp;&emsp;<a href="#41">  # 1*8=8 2*8=16 3*8=24 4*8=32 5*8=40 6*8=48 7*8=56 8*8=64</a><br>
&emsp;&emsp;<a href="#42">  # 1*9=9 2*9=18 3*9=27 4*9=36 5*9=45 6*9=54 7*9=63 8*9=72 9*9=81</a><br>
&emsp;&emsp;<a href="#43">Python 参数传递</a><br>
&emsp;&emsp;<a href="#44">__prepare__ 方法</a><br>
&emsp;&emsp;<a href="#45">__init__, __new__ 区别</a><br>
&emsp;&emsp;<a href="#46">with 实现原理</a><br>
&emsp;&emsp;<a href="#47">Python 垃圾回收机制</a><br>
&emsp;&emsp;<a href="#48">__iter__和__next__</a><br>
&emsp;&emsp;<a href="#49">yield 关键字作用</a><br>
&emsp;&emsp;<a href="#50">await 和 async 等关键字，版本，作用</a><br>
&emsp;&emsp;<a href="#51">可变对象和不可变对象</a><br>
&emsp;&emsp;<a href="#52">怎么实现单例</a><br>
&emsp;&emsp;<a href="#53">super 作用，超类</a><br>
&emsp;&emsp;<a href="#54">property 作用</a><br>
&emsp;&emsp;<a href="#55">slot 作用</a><br>
&emsp;&emsp;<a href="#56">生成器原理</a><br>
&emsp;&emsp;<a href="#57">什么是python的生成器?</a><br>
&emsp;&emsp;<a href="#58">tornado 实现原理</a><br>
&emsp;&emsp;<a href="#59">flask 源码</a><br>
&emsp;&emsp;<a href="#60">介绍一下Python 锁？</a><br>
&emsp;&emsp;<a href="#61">代码题:实现一个生产者消费者</a><br>
# <a name="0">Python 中的整数</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. [《深度剖析CPython解释器》5. 解密Python中的整数在底层是如何实现的，以及为什么Python中大整数的运算不会溢出](https://www.cnblogs.com/traditional/p/13437243.html)

    Python 中整数对应的值在底层是使用数组来存储的，所以理论上永远不会溢出。

# <a name="1">Dict 底层实现原理</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="2">Python List 底层实现原理</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="3">Python 中的小整数池</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    为了避免大量对象的创建和销毁，Python 将那些使用频率高的整数预先创建好，
    这些预先创建好的整数会放在一个静态数组里面，称为小整数对象池。如果需要使用的话会直接拿来用，而不用重新创建。

# <a name="4">def func(a, b=[]) 这种写法有什么坑？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="5">is 和 == 的区别</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    is 比较的是两个实例对象是不是完全相同，它们是不是同一个对象，占用的内存地址是否相同，相当于调用 id() 方法。
    == 比较的是两个对象的内容是否相等，即内存地址可以不一样，内容一样就可以了，默认会调用对象的 __eq__()方法。

# <a name="6">Python 递归的最大层数？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    默认是 1000
    可以通过 sys.setrecursionlimit() 进行设置

# <a name="7">a = [1, 2, 3, 4, 5]，b = a 和 b = a[:]，有区别么？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    前者传递引用，后者是拷贝

# <a name="8">Python 中的 MRO</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="9">静态方法和类方法，普通方法</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="10">Python 的元类</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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
    # <a name="11">不使用 class 关键字来定义，而使用 type，如下：</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="12">__call__ 方法</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    object() 相当于 object.__call__()
    __call__() 的作用是使实例能够像函数一样被调用，
    同时不影响实例本身的生命周期（__call__()不影响一个实例的构造和析构）。

    ```
    class Foo:
        def __call__(self, a, b, c):
            # <a name="13">...</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    x = Foo()
    x(1, 2, 3) # __call__
    ```

# <a name="14">OrderedDict 怎么实现</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    在继承了自身的 Dict 之后，类中还声明了一个循环双向链表 self.__root 作为自己的属性，
    用另一个 Dict 结构 self.__map 来存储 Key 到链表节点的对应关系，
    可以类比 LRU 缓存的实现方法（Leetcode）。

    Python 3.7 中原生 Dict 与 OrderedDict 的区别：
        1. OrderedDict 支持对 Key 的顺序有关的操作，
           比如说把某个 Key 挪到头部或者尾部，
           逆序给出 Key 的序列（这个特性在 Python 3.8 中加到了原生 Dict 中）等；
        2. OrderedDict 比较的时候会把 Key 的插入顺序考虑进去，而 Dict 不会

# <a name="15">Python 线程安全</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

# <a name="16">Python 的 sort 函数实现原理</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. [python sort函数内部实现原理](https://www.cnblogs.com/clement-jiao/p/9243066.html)

    内部实现机制为：Timesort
    最坏时间复杂度为：O（n log n）
    空间复杂度为：O（n）
    算法原理：
       Timsort 是结合了合并排序（merge sort）和插入排序（insertion sort）而得出的排序算法，它在现实中有很好的效率。
       该算法找到数据中已经排好序的块-分区，每一个分区叫一个 run，然后按规则合并这些 run。

# <a name="17">xrange() 和 range() 的区别</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    Python2 中 xrange() 返回生成器，range() 返回 list，
    Python3 中没有 xrange()，只有 range()，并且返回生成器。

# <a name="18">Python 中的 GIL</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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
        对于 CPU 密集型代码，使用多进程，或者改为使用 C 实现
        对于 IO 密集型代码，使用多线程

# <a name="19">Python 中的 __dict__ 和 dir()</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. [what's the biggest difference between dir and __dict__ in python](https://stackoverflow.com/questions/14361256/whats-the-biggest-difference-between-dir-and-dict-in-python)

    dir() 不仅查找对象的 __dict__（有时甚至不存在），它还会根据对象的继承关系提供所有可用属性。
    __dict__ 只是获取实例的“本地”属性集，并不包含该实例的所有可用属性。

# <a name="20">list() 和 [] 哪个效率更快？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    创建空列表时，[] 要比 list() 快。

    对于 []，它是 Python 中的一组字面量（literal），像数字之类的字面量一样，表示确切的固定值。
    也就是说，Python 在解析到它时，就知道它要表示一个列表，因此会直接调用解释器中构建列表的方法来创建列表。

    对于 list()，“list”只是一个普通的名称，并不是字面量，也就是说解释器一开始并不认识它。
    因此，解释器的第一步是要找到这个名称（对应 LOAD_NAME）。它会按照一定的顺序，
    在各个作用域中逐一查找（局部作用域--全局作用域--内置作用域），直到找到为止，找不到则会抛出NameError。
    第二步是把这个名称当作可调用对象来调用，即把它当成一个函数进行调用（对应 CALL_FUNCTION）。
    因此，list() 在创建列表时，需要经过名称查找与函数调用两个步骤，才能真正开始创建列表。

    因为 list() 涉及的执行步骤更多，因此它比 [] 要慢一些。

# <a name="21">列表推导式，字典推导式，集合推导式</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    列表推导式：
        ```
        # <a name="22">使用 [] 生成 list</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
        a = [i for i in range(10)]
        # <a name="23">[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

        # <a name="24">使用 () 生成 generator</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
        a = (i for i in range(10))
        # <a name="25"><generator object <genexpr> at 0x00000281C61DA840></a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
        ```

    字典推导式：
        ```
        a = {'a': 1, 'b': 2, 'c': 3}
        b = {v: k for k, v in a.items()}
        # <a name="26">{1: 'a', 2: 'b', 3: 'c'}</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
        ```

    集合推导式：
        ```
        a = {x**2 for x in [1, 1, 2]}
        # <a name="27">{1, 4}</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
        ```

# <a name="28">Python 装饰器</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

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

        # <a name="29">foo = use_logging(foo)</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
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

        # <a name="30">@use_logging(level="warn") 等价于 @decorator</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
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

        # <a name="31">它的执行顺序是从里到外，最先调用最里层的装饰器，最后调用最外层的装饰器，它等效于</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
        # <a name="32">f = a(b(c(f)))</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
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

# <a name="33">一行代码写出乘法表</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    ```
    print("\n".join([" ".join(["{}*{}={}".format(j, i, j*i) for j in range(1, i+1)]) for i in range(1, 10)]))

    # <a name="34">1*1=1</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="35">1*2=2 2*2=4</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="36">1*3=3 2*3=6 3*3=9</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="37">1*4=4 2*4=8 3*4=12 4*4=16</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="38">1*5=5 2*5=10 3*5=15 4*5=20 5*5=25</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="39">1*6=6 2*6=12 3*6=18 4*6=24 5*6=30 6*6=36</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="40">1*7=7 2*7=14 3*7=21 4*7=28 5*7=35 6*7=42 7*7=49</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="41">1*8=8 2*8=16 3*8=24 4*8=32 5*8=40 6*8=48 7*8=56 8*8=64</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    # <a name="42">1*9=9 2*9=18 3*9=27 4*9=36 5*9=45 6*9=54 7*9=63 8*9=72 9*9=81</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    ```

# <a name="43">Python 参数传递</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>
    参考：
        1. [Python的函数参数传递：传值？引用？](http://winterttr.me/2015/10/24/python-passing-arguments-as-value-or-reference/)
        2. [关于Python中参数传递和作用域的问题？ - Coldwings的回答 - 知乎](https://www.zhihu.com/question/58539447/answer/157433659)

    在 Python 中，类型属于对象，变量是没有类型的。
    严格意义上来说 Python 传参策略既不是 call-by-reference，也不是 call-by-value，而是另一种机制 call-by-object，
    亦 call-by-Object-reference ，或 call-by-sharing（参数传递是对一个对象的引用，但是这个引用却又是 passed by value）。
    可以简单理解为实际上参数传递的是值，但是，这个「值」是对对象的引用。

# <a name="44">__prepare__ 方法</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. https://docs.python.org/3/reference/datamodel.html#preparing-the-class-namespace

    用于创建类的命名空间。

    参数要求：
        __prepare__ 方法的第一个参数是元类，随后两个参数分别是要构建的类的名称和基类组成的元组，返回值必须是映射。

    工作原理：
        元类构建新类时，解释器会先调用 __prepare__ 方法，使用类定义体中的属性创建映射。
        接着把 __prepare__ 方法返回的映射会传给 __new__ 方法的最后一个参数，然后再传给 __init__ 方法。

    使用场景：
        __prepare__ 只在元类中有用，而且必须声明为类方法（即要使用 @classmethod 装饰器定义）。

        3.6 版本以前，prepare 方法主要用来返回一个 OrderDict 对象，以保存类中属性的添加顺序。
        而 3.6 版本以后，默认已经是保持顺序的了。

# <a name="45">__init__, __new__ 区别</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    Python2 中的经典类是没有 __new__ 方法的。
    类（必须是新式类）在实例化的时候会先调用 __new__，然后调用 __init__。

    __new__:
        为实例对象申请一片内存，第一个参数是 cls（代表类对象本身），返回创建后的实例，
        这个返回值会当做 __init__ 的第一个参数;
    __init__:
        为实例对象绑定属性，第一个参数是 self（代表实例对象本身），没有返回值;

# <a name="46">with 实现原理</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    with 在执行的时候，会执行紧随其后的代码，并调用该对象的 __enter__ 方法，执行相关操作，
    如果出现异常，或者代码执行结束，则会自动调用 __exit__ 方法，进行资源的清理。

    __enter__:
        进入对象的上下文管理器调用的方法，会返回一个值，并赋值给 as 关键词之后的变量
    __exit__:
        退出对象的上下文管理器调用的方法，定义了处理结束后要做的事情，比如文件的关闭，socket 的断开等

# <a name="47">Python 垃圾回收机制</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    参考：
        1. https://devguide.python.org/garbage_collector/
        2. https://www.jianshu.com/p/1e375fb40506
        3. https://juejin.cn/post/6844903629556547598

    引用计数：
        在 Python 中每一个对象的核心就是一个结构体 PyObject，它的内部有一个引用计数器。
        被引用了加 1，引用它的对象被删除的时候减 1，直到当它的引用计数变为 0 的时候，垃圾回收机制就会将它回收。
        缺点：
            1. 执行效率低，维护引用计数机制会带来额外操作
            2. 存在循环引用问题（只有容器对象才存在）

    标记-清除：
        主要用于识别循环引用。
        维护两个链表，一个是 root 链表（root object），另外一个是 unreachable 链表。
        从 root 链表开始遍历，经过一系列算法，最终需要回收的对象会放到 unreachable 链表里。
        为什么移动的是需要清除的对象，而不是不需要清除的对象？
            通常情况下，需要清除的对象相较于不需要清除的对象数量少很多。

    分代回收：
        分代回收思想将对象分为三代（generation 0,1,2），0 代表幼年对象，1 代表青年对象，2 代表老年对象。
        根据弱代假说（越年轻的对象越容易死掉，老的对象通常会存活更久），
        新生的对象被放入 0 代，如果该对象在第 0 代的一次 GC 垃圾回收中活了下来，
        那么它就被放到第 1 代里面（它就升级了）。
        如果第 1 代里面的对象在第 1 代的一次 GC 垃圾回收中活了下来，它就被放到第 2 代里面。

        从上一次第 0 代 GC 后，如果分配对象的个数减去释放对象的个数大于 threshold0，
        那么就会对第 0 代中的对象进行 GC 垃圾回收检查。
        从上一次第 1 代 GC 后，如果第 0 代被 GC 垃圾回收的次数大于 threshold1，
        那么就会对第 1 代中的对象进行 GC 垃圾回收检查。
        同样，从上一次第 2 代 GC 后，如果第 1 代被 GC 垃圾回收的次数大于 threshold2，
        那么就会对第 2 代中的对象进行 GC 垃圾回收检查。

# <a name="48">__iter__和__next__</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo
    https://zhuanlan.zhihu.com/p/266512848

# <a name="49">yield 关键字作用</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="50">await 和 async 等关键字，版本，作用</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="51">可变对象和不可变对象</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="52">怎么实现单例</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="53">super 作用，超类</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="54">property 作用</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="55">slot 作用</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="56">生成器原理</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="57">什么是python的生成器?</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="58">tornado 实现原理</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="59">flask 源码</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="60">介绍一下Python 锁？</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

# <a name="61">代码题:实现一个生产者消费者</a><a style="float:right;text-decoration:none;" href="#index">[Top]</a>

    todo

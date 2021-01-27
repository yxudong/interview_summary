#### def func(a,b=[]) 这种写法有什么坑？

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

#### is 和 == 的区别

    is 比较的是两个实例对象是不是完全相同，它们是不是同一个对象，占用的内存地址是否相同，相当于调用 id() 方法。
    == 比较的是两个对象的内容是否相等，即内存地址可以不一样，内容一样就可以了，默认会调用对象的 __eq__()方法。

#### Python dict 底层实现原理

    Python 3.6 之前维护一张 hash 表，字典是无序的
    Python 3.7 之后同时维护 hash 表和数组，字典是有序的

    如何处理 hash 冲突？
        使用开放地址法中的线性探测

#### Python list 底层实现原理

    数组，每个位置存放的是指向实际对象的指针

#### Python 递归的最大层数？

    默认是 1000
    可以通过 sys.setrecursionlimit() 进行设置


什么是面向对象的mro
多个装饰器的执行顺序
静态方法和类方法，普通方法
Python 的元类
__init__, __new__ 区别
怎么理解 __call__
Python 垃圾回收机制
order_dict 怎么实现，3.8 开始字典有序
yield 关键字作用
await 和 async 等关键字，版本，作用
tornado 实现原理
可变对象和不可变对象
GIL
怎么实现单例
super 作用
列表推导式，字典推导式，生成器推导式
list() 和 [] 哪个效率更快？
property 作用
__dict__ 内容
slot 作用
介绍一下Python 锁？
什么是python的生成器?
装饰器的写法以及应用场景。
什么是反射？以及应用场景？
面向对象深度优先和广度优先是什么？
经典类和新式类
a=[1,2,3,4,5]，b=a和b=a[:]，有区别么？
 *arg和**kwarg作用
xrange和range的区别
Python 小整数池
Python 的 sort 函数实现原理
Python 装饰器的原理
一行代码写出乘法表
print("\n".join([" ".join(["%s*%s=%-2s"%(j,i,j*i) for j in range(1,i+1)]) for i in range(1,10)]))
Python dict，set，list，tuple 底层实现原理，源码
代码题:实现一个生产者消费者
















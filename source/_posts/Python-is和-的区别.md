---
title: 'Python:is和==的区别'
date: 2018-04-16 21:08:08
tags:
    - Python
categories:
    - Python
---
我们先来看几个例子：
```
a = "hello"
b = "hello"
print(a is b)  # 输出 True 
print(a == b)  # 输出 True

a = "hello world"
b = "hello world"
print(a is b)  # 输出 False
print(a == b)  # 输出 True 

a = [1, 2, 3]
b = [1, 2, 3]
print(a is b)  # 输出 False
print(a == b)  # 输出 True 

a = [1, 2, 3]
b = a
print(a is b)  # 输出 True 
print(a == b)  # 输出 True
```
上面的输出结果中为什么有的 is 和 == 的结果相同，有的不相同呢？我们来看下官方文档中对于 is 和 == 的解释。
官方文档中说 is 表示的是对象标示符（object identity），而 == 表示的是相等（equality）。is 的作用是用来检查对象的标示符是否一致，也就是比较两个对象在内存中的地址是否一样，而 == 是用来检查两个对象是否相等。
我们在检查 a is b 的时候，其实相当于检查 id(a) == id(b)。而检查 a == b 的时候，实际是调用了对象 a 的 __eq()__ 方法，a == b 相当于 a.__eq__(b)。
一般情况下，如果 a is b 返回True的话，即 a 和 b 指向同一块内存地址的话，a == b 也返回True，即 a 和 b 的值也相等。
好了，看明白上面的解释后，我们来看下前面的几个例子。
一般情况下，如果 a is b 返回True的话，即 a 和 b 指向同一块内存地址的话，a == b 也返回True，即 a 和 b 的值也相等。
好了，看明白上面的解释后，我们来看下前面的几个例子。
```
a = "hello"
b = "hello"
print(id(a))   # 输出 140506224367496
print(id(b))   # 输出 140506224367496
print(a is b)  # 输出 True 
print(a == b)  # 输出 True

a = "hello world"
b = "hello world"
print(id(a))   # 输出 140506208811952
print(id(b))   # 输出 140506208812208
print(a is b)  # 输出 False
print(a == b)  # 输出 True 

a = [1, 2, 3]
b = [1, 2, 3]
print(id(a))   # 输出 140506224299464
print(id(b))   # 输出 140506224309576
print(a is b)  # 输出 False
print(a == b)  # 输出 True 

a = [1, 2, 3]
b = a
print(id(a))   # 输出 140506224305672
print(id(b))   # 输出 140506224305672
print(a is b)  # 输出 True 
print(a == b)  # 输出 True
```
打印出 id(a) 和 id(b) 后就很清楚了。只要 a 和 b 的值相等，a == b 就会返回True，而只有 id(a) 和 id(b) 相等时，a is b 才返回 True。
这里还有一个问题，为什么 a 和 b 都是 "hello" 的时候，a is b 返回True，而 a 和 b都是 "hello world" 的时候，a is b 返回False呢？
这是因为前一种情况下Python的字符串驻留机制起了作用。对于较小的字符串，为了提高系统性能Python会保留其值的一个副本，当创建新的字符串的时候直接指向该副本即可。所以 "hello" 在内存中只有一个副本，a 和 b 的 id 值相同，而 "hello world" 是长字符串，不驻留内存，Python中各自创建了对象来表示 a 和 b，所以他们的值相同但 id 值不同。
总结一下，is 是检查两个对象是否指向同一块内存空间，而 == 是检查他们的值是否相等。可以看出，is 是比 == 更严格的检查，is 返回True表明这两个对象指向同一块内存，值也一定相同。
看到这里，大家是不是搞懂了 is 和 == 的区别呢？
那我们深入一步来思考一下下面这个问题：
   Python里和None比较时，为什么是 is None 而不是 == None 呢？
这是因为None在Python里是个单例对象，一个变量如果是None，它一定和None指向同一个内存地址。而 == None背后调用的是__eq__，而__eq__可以被重载，下面是一个 is not None但 == None的例子
```
class Foo(object):
    def __eq__(self, other):
        return True

f = Foo()
print(f == None)  # 输出 True
print(f is None)  # 输出 False
```
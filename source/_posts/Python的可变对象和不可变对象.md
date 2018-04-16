---
title: Python的可变对象和不可变对象
date: 2018-04-16 20:37:16
tags:
    - Python
categories:
    - Python
---
Python中有可变对象和不可变对象之分。可变对象创建后可改变但地址不会改变，即变量指向的还是原来的变量；不可变对象创建之后便不能改变，如果改变则会指向一个新的对象。   
Python中dict、list是可变对象，str、int、tuple、float是不可变对象。  
来看一个字符串的例子  
```
a = "hello"
print(id(a))  # 输出 140022851974560

a[0]="a"  # 抛出异常：TypeError: 'str' object does not support item assignment

a = a + " world"
print(id(a))  # 输出 140022850763824
```
上面的例子里，修改a指向的对象的值会导致抛出异常。
执行 a = a + " world"时，先计算等号右边的表达式，生成一个新的对象赋值到变量a，因此a指向的对象发生了改变，id(a) 的值也与原先不同。
再来看一个列表的例子
```
a = [1,2,3]
print(id(a))  # 输出 140022851303976

a[0]=5
print(a)      # 输出 [5, 2, 3]
print(id(a))  # 输出 140022851303976

a.append(5)
print(a)      # 输出 [5, 2, 3, 5]
print(id(a))  # 输出 140022851303976

b = a
print(id(b))  # 输出 140022851303976

b[0] = 6
print(b)      # 输出 [6, 2, 3, 5]
print(a)      # 输出 [6, 2, 3, 5]

c = b[:]
print(id(c))  # 输出 140022851006760
print(id(b))  # 输出 140022851303976

c.append(7)
print(c)      # 输出 [6, 2, 3, 5, 7]
print(b)      # 输出 [6, 2, 3, 5]
```
上面的例子里，修改a指向的对象的值会导致抛出异常。
执行 a = a + " world"时，先计算等号右边的表达式，生成一个新的对象赋值到变量a，因此a指向的对象发生了改变，id(a) 的值也与原先不同。
再来看一个列表的例子
```
a = [1,2,3]
print(id(a))  # 输出 140022851303976

a[0]=5
print(a)      # 输出 [5, 2, 3]
print(id(a))  # 输出 140022851303976

a.append(5)
print(a)      # 输出 [5, 2, 3, 5]
print(id(a))  # 输出 140022851303976

b = a
print(id(b))  # 输出 140022851303976

b[0] = 6
print(b)      # 输出 [6, 2, 3, 5]
print(a)      # 输出 [6, 2, 3, 5]

c = b[:]
print(id(c))  # 输出 140022851006760
print(id(b))  # 输出 140022851303976

c.append(7)
print(c)      # 输出 [6, 2, 3, 5, 7]
print(b)      # 输出 [6, 2, 3, 5]
```
上面对a修改元素、添加元素，变量a还是指向原来的对象。
将a赋值给b后，变量b和a都指向同一个对象，因此修改b的元素值也会影响a。
变量c是对b的切片操作的返回值，切片操作相当于浅拷贝，会生成一个新的对象，因此c指向的对象不再是b所指向的对象，对c的操作不会改变b的值。
理解了上面不可变对象和可变对象的区别后，我们再来看一个有趣的问题
```
class Group(object):
    def __init__(self, group_id, members=[]):
        self.group_id = group_id
        self.members = members
    def add_member(self, member):
        self.members.append(member)

group1 = Group(1)
group1.add_member("Wang")
group1.add_member("Sun")
print(id(group1))        # 输出 140280211197776
print(group1.members)    # 输出 ['Wang', 'Sun']

group2 = Group(2)
group2.add_member("Zhang")
group2.add_member("Li")

print(id(group2))        # 输出 140280211197840 
print(group1.members)    # 输出 ['Wang', 'Sun', 'Zhang', 'Li']
print(group2.members)    # 输出 ['Wang', 'Sun', 'Zhang', 'Li']
```
明明group1和group2是不同的对象（id值不同），为什么调用group2的add_member方法会影响group1的members？
其中的奥妙就在于__init__函数的第二个参数是默认参数，默认参数的默认值在函数创建的时候就生成了，每次调用都是用了这个对象的缓存。我们检查id(group1.mebers)和id(group2.members)，可以发现他们是相同的
```
print(id(group1.members))  # 输出 140127132522040
print(id(group2.members))  # 输出 140127132522040
```
所以，group1.members和group2.members指向了同一个对象，对group2.members的修改也会影响group1.members。  
好了，问题来了，怎样改代码才能解决上面默认参数的问题呢？  
其实很简单，只要传入None作为默认参数，在创建对象的时候动态生成列表，如下  
```
class Group(object):
    def __init__(self, group_id, members=None):
        self.group_id = group_id
        if members is None:
            self.members = []

    def add_member(self, member):
        self.members.append(member)
```  
这样对于不同的group对象，它们的members也是不同的对象，所以不会再出现更新一个group对象的members也会更新另外一个group对象的members了。
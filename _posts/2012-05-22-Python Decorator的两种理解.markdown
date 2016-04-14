---
layout:     post
title:      "Python Decorator的两种理解"
date:       2012-05-22 20:08
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - python
---

# 1、函数作为Decorator

## 1) 概述
装饰器函数是一种比较常见的方式，表现出来是一个函数内嵌套定义另一个函数。这也是Python特有的语法。

```python
	
#定义一个Decorator，表面上，它与函数无异，但它的参数也是一个函数。
def deco(func):
	#这个wrap函数是对原始函数func()的一个封装。
	def wrap(*args, **kargs):
		#定义封装后的函数的行为
		print 'exec before func...'
		func(*args, **kargs)
		print 'exec after func...'
	#返回封装好的新函数
	return wrap
```
		
## 2) 实例
《Head First Design Pattern》一书中对装饰墙有一个很浅显的调咖啡的例子。我们可以尝试使用Python装饰器实现一次。

### A) 需求描述
Starbuzz商店卖咖啡，咖啡本身有多种，包括但不限于：HouseBlend、DarkRoast、Decaf、Espresso，不同的咖啡价格不同；同时，客户也可以选择在这些咖啡中加入各种调料，例如：Steamed Milk、Soy、Mocha等等，当然添加调料也是要加钱的。

现在需要的是为Starbuzz做一个订餐系统，而其核心部分则是为这一套咖啡模型建模。书中有了很好的解释和基于Java的实现，并引出了一个原则：“open-close”原则，即“对扩展开放，对修改关闭。”

## B) 实现
这里仅讨论Python的实现,咖啡的种类使用继承关系，咖啡调料使用装饰器：

```python
	
def steamed_milk(func):
    '''调料：牛奶'''
    def wrapper(factory, beverage):
        func(factory, beverage)
        beverage.cost += 1
        beverage.desc = '%s and add milk.' %beverage.desc
        return beverage
    return wrapper

def soy(func):
    '''调料：沙拉'''
    def wrapper(factory, beverage):
        func(factory, beverage)
        beverage.cost += 0.5
        beverage.desc = '%s and add soy.' %beverage.desc
        return beverage
    return wrapper

class Beverage(object):
    '''原始的咖啡饮料'''
    desc = 'Beverage.'
    
    def __init__(self, cost=0.99):
        self.cost = cost

class HouseBlend(Beverage):
    '''HouseBlend咖啡'''
    desc = 'HouseBlend.'
    
    def __init__(self,cost=1.99):
        '''这里顺便示范了两种初始化父类的方法，这是其一'''
        super(HouseBlend, self).__init__(cost)

class DarkRoast(Beverage):
    '''DarkRoast咖啡'''
    desc = 'DarkRoast.'
    
    def __init__(self,cost=2.99):
        '''这里顺便示范了两种初始化父类的方法，这是其二'''
        Beverage.__init__(self, cost)


class CoffeeFactory(object):
    '''咖啡工厂，用于创建各种咖啡'''
    @steamed_milk
    def make_milk_coffee(self, beverage):
        '''创建牛奶咖啡'''
        print 'making milk coffee...'
        return Beverage(beverage.cost)

    @soy
    @steamed_milk
    def make_milk_soy_coffee(self, beverage):
        '''创建牛奶萨拉咖啡'''
        print 'making milk soy coffee...'
        return Beverage(beverage.cost)


f = CoffeeFactory()
h = HouseBlend()
milk_house_blend = f.make_milk_coffee(h)
print 'Done,Here is the %s, the cost is $%s' %(
    milk_house_blend.desc, milk_house_blend.cost
)

d = DarkRoast()
milk_soy_dark_roast = f.make_milk_soy_coffee(d)
print 'Done,Here is the %s, the cost is $%s' %(
    milk_soy_dark_roast.desc, milk_soy_dark_roast.cost
)
```

当然这种方式看起来还是不免有些奇怪，这些装饰器都是静态地标注在方法定义上的，而不是动态的在调用的时候来指定。多少显得有些累赘。当然我们也可以以Java装饰器的方式，即通过一层层调用的方式来实现这个需求，那样可能更直接。

但是换一种场景，Python装饰器就相当使用，可以非常方便地实现面向方面编程。例如线程同步，日志打印等等。

```python
	
def sync(func):
'''同步代码块'''
def wrapper(*args, **kv):
    self = args[0]
    self.lock.acquire()
    try:
        return func(*args, **kv)
    finally:
        self.lock.release()
return wrapper
```

---
# 2、类作为Decorator

基于类也可以实现装饰器，之前看过一片文章，认为基于类的装饰器更为“正统”和容易理解。个人感觉理解了以后（就装饰器本身而言）都不难理解，Python语法灵活，这只是第二条路罢了。 :)

对于class作为装饰器，实际上是调用了class的\_\_call__()方法。

```python

class SomeDecorator(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print 'instance %s of class %s this is now decorated! method of instance is %s' % (
            self.obj, self.cls, self.func
        )
        return self.func.__call__(self, *args, **kwargs)

    def __get__(self, instance, owner):
        self.cls = owner
        self.obj = instance

        return self.__call__

class SomeClass(object):
    @SomeDecorator
    def dostuff(self, foo, bar):
        print 'do %s, %s' % (foo, bar)
        
sc = SomeClass()
sc.dostuff("foo", 'bar')
```

---

# 参考
1. 《[Decorators for Functions and Methods](http://www.python.org/dev/peps/pep-0318/)》
2. 《[Python: Decorate a Method That Gets Passed the Class Instance](http://christiankaula.com/python-decorate-method-gets-class-instance.html)》
3. 《[Python Gossip: 描述器](http://caterpillar.onlyfun.net/Gossip/Python/Descriptor.html)》
4. 《[Python Data Model](http://docs.python.org/reference/datamodel.html?highlight=__call__#implementing-descriptors)》
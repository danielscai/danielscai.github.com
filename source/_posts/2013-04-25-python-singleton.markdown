---
layout: post
title: "python 设计模式之单例 singleton"
date: 2013-04-25 12:52
comments: true
categories: coding
---

{% img right /images/main/python-logo.gif 250 150 'openstack' 'openstack' %}
python单例实现大概有3种，各有优缺点


* 装饰器

优点：直接添加在类定义代码上面，比多重继承更加直观
缺点： 类调用的时候 myclass()是一个单例对象，但是myclass本身变成了一个方法，而不是一个类，因此不能调用这个类的类方法

<!-- more -->

* 基类

优点：相比装饰器，这是一个真正的类
缺点： 需要使用多重继承， `__new__`方法可能会被重写，单例实现可能会被覆盖

* 元类

优点：是一个真正的类，达到了多重继承的效果，而不会被重写
缺点：相对来说没有其他两个清晰，但是还是比较清晰的

使用metaclass实现单例，
示例代码

    class Singleton(type):
        _instances = {}
        def __call__(cls, *args, **kwargs):
            if cls not in cls._instances:
                cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
            return cls._instances[cls]
    
    class MyClass(BaseClass):
        __metaclass__ = Singleton

**代码分析**

定义了一个Singleton类，
使用类变量_instances字典记录类是否存在，
该类实现了一个 `__call__`方法查询类是否在类变量`_instances中`，如果存在，直接返回改类，如果不存在，使用super生成一个该类，并记录到类变量`_instances`中


参考
[http://stackoverflow.com/questions/6760685/creating-a-singleton-in-python](http://stackoverflow.com/questions/6760685/creating-a-singleton-in-python)


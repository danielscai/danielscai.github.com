---
layout: post
title: "python 设计模式之 装饰器 "
date: 2013-05-08 22:40
comments: true
categories: coding
---

{% img right /images/main/python-logo.gif 250 150 'python' 'python' %}

之前介绍过 python 的[单例](http://dnscai.com/blog/2013/04/25/python-singleton/)


其实python 不需要单例 (为什么？ 想一想，模块不就是最好的单例么？)


本文主要讨论的是python 的装饰器，flask 的路由就是使用装饰器实现的，flask介绍可以参考 [http://dnscai.com/blog/2013/04/25/python-flask-web-framework-intro/](http://dnscai.com/blog/2013/04/25/python-flask-web-framework-intro/)

<!--more-->
下面是一段简单的flask路由规则

```
@vm.route("/<management_zone>/<availability_zone>/<string:x_id>",
    methods=[ 'GET','POST','PUT','DELETE'])
def vm(management_zone,availability_zone,x_id):
    return ''

```

**复用函数问题**

有些函数可能是需要在调用没一个url的时候都需要执行，比如说认证模块。
传统的解决方案就是写一个认证函数，在每一个url函数下调用这个函数。

这的确可以解决问题，但是代码不能优雅一点么，不能更pythonic一点么？ 

**简单装饰器**

装饰器的作用很简单，就是在运行某个函数之前，运行一些其他代码，作用如下

```
foo=bar(foo)
```

就像foo函数被bar装饰过了一样，python中提供了一个装饰器的语法糖，'@'

让我们来看看如何使用装饰器写一个认证函数


**认证函数 装饰器**

下面代码生成一个`auth_deco`的函数，这个函数特别之处在于他会返回一个传入的函数，在返回之前，做了认证的工作

```
def auth_deco(func):
    '''
    ladp auth decorator
    '''
    @functools.wraps(func)
    def wrapper(*argv,**kwgs):
        username=request.headers.get('username')
        password=request.headers.get('password')
        if password != None:
            password=base64.decodestring(password)
        _ldap_auth(username,password)
        return func(*argv,**kwgs)
    return wrapper

```

**使用这个装饰器**

在`@vm.route`的下面添加`@auth_deco`就大共告成，而且非常pythonic

```
@vm.route("/<management_zone>/<availability_zone>/<string:x_id>",
    methods=[ 'GET','POST','PUT','DELETE'])
@auth_deco
def vm(management_zone,availability_zone,x_id):
    return ''
```

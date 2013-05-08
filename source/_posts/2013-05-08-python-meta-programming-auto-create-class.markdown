---
layout: post
title: "python 元编程之 自动生成类"
date: 2013-05-08 23:06
comments: true
categories: coding
---

{% img right /images/main/python-logo.gif 250 150 'python' 'python' %}

元编程是动态语言最大的特点，却是一个经常被程序员忽略的技巧。

元编程对初学者来讲，稍显复杂。那么元编程到底可以给我们带来什么特异功能呢？

下面试用项目中用到的自动生成类的例子讲解元编程 

<!--more-->

**需求**

这段代码主要是生成项目中的http错误类，用来返回对应的http code和一段封装成json的错误提示信息

**代码**

先来看下代码

1. 将错误类的类名和对应的错误码记录到一个字典中
---
下面字典的值，都是我已经定义好的，比如`BAD_REQUEST`就是400 
```
except_dict={
    'ActionNotSupportedError': ACTION_NOT_SUPPORTED,
    'BadRequestError':BAD_REQUEST,
    'NotFoundError':NOT_FOUND,
    'UnauthorizedError':UNAUTHORIZED,
    'ConflictError':CONFLICT,
    'ParamsMissingError': PARAMS_MISSING,
    'OverlimitError': OVERLIMIT,
    'OSApiError': OSAPI_ERROR,
    'MethodNotAllowedError': METHOD_NOT_ALLOWED,
    'InternalServerError': INTERNAL_SERVER_ERROR,
}

```

2. 定义类的函数
---

这里定义了两个函数,一个是`__init__`和`__str__` ，然后将其写入属性字典`attr`

```
def __init__(self,errmsg):
    ''' make returned error message'''
    self.value='{"error_message":"%s"}' % errmsg
    zlog.logger("Exception: "+self.value)

def __str__(self):
    return repr(self.value)

exceptions_list=[]
bases=(Exception,)
attrs={
    '__init__':__init__,
    '__str__':__str__,
}


```

3. 生成类
---
最后一步，也是最重要的一步，遍历第1步中生成的字典`except_dict`,属性code 设置为字典对应key的值
使用`type`生成一个类，使用globals()将类注册到当前模块空间
```
for (eklass_name,code) in except_dict.items():
    attrs['code']=code
    eklass=type(eklass_name,bases,attrs)
    exceptions_list.append(eklass)
    globals().update( {eklass_name: eklass } )

```

这样一段扩展性非常高的错误处理代码就完成了，新添加一个错误码，只需要添加一行`except_dict`字典值，剩下的代码会帮你搞定。



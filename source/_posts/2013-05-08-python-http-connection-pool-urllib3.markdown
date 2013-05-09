---
layout: post
title: "python http connection pool urllib3"
date: 2013-05-08 19:39
comments: true
categories: coding
---

{% img right /images/main/python-logo.gif 250 150 'python' 'python' %}
随着项目越来越庞大，直接http请求成为不可忽略的一个问题，需要有一种机制来管理和复用http链接，http连接池就是设计来解决这个问题的.

连接池是创建和管理一个连接的缓冲池的技术，这些连接准备好被任何需要它们的线程使用

使用连接池有如下好处

<!--more-->

**减少连接创建时间**  

http连接需要完成3次握手，建立一个连接，如果这类连接是“循环”使用的，使用该方式这些花销就可避免。


**简化的编程模式**  

当使用连接池时，每一个单独的线程能够像创建了一个自己的http连接一样操作，允许用户直接使用从池中得到的连接编程。


**控制资源使用**  

如果用户不使用连接池，而是每当线程需要时创建一个新的连接，那么用户的应用程序的资源使用会产生非常大的浪费并且可能会导致高负载下的异常发生。
连接池能够使性能最大化，同时还能将资源利用控制在一定的水平之下，如果超过该水平，应用程序将崩溃而不仅仅是变慢。

python 中可以使用`urllib3`来使用http 连接池

1. 安装
---

使用 pip

    pip install urllib3

或者clone最新的代码 [github.com/shazow/urllib3.](github.com/shazow/urllib3.)

2. 使用
---

```
>>> import urllib3
>>> http = urllib3.PoolManager()
>>> r = http.request('GET', 'http://google.com/')
>>> r.status
200
>>> r.headers['server']
'gws'
>>> r.data
...
```

3. 使用 connection from url
---

处理hostname有时候是一件非常恼人的事情，去除http头，得到端口等等。
好在urllib3帮我们都处理了这些累活，使用connection from url 即可

```
>>> conn = urllib3.connection_from_url('http://google.com')
>>> r1 = conn.request('GET', 'http://google.com/')
>>> r2 = conn.request('GET', '/mail')
>>> r3 = conn.request('GET', 'http://yahoo.com/')
Traceback (most recent call last)
  ...
HostChangedError: Connection pool with host 'http://google.com' tried to
open a foreign host: http://yahoo.com/
```

**注意**
如果代码是从httplib转过来的，httplib中的request函数和urllib3中的request函数是不一样的，需要使用 urllib3中的urlopen来替换以前的request函数

这个比较坑爹，我花了很长时间在这个pitfall上面

```
>>> conn = urllib3.connection_from_url('http://google.com')
>>> r1 = conn.urlopen('GET', 'http://google.com/')
```

参考 [http://urllib3.readthedocs.org/en/latest/](http://urllib3.readthedocs.org/en/latest/)

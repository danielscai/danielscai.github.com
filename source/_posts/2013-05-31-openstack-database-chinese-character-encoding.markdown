---
layout: post
title: "openstack 数据库中文乱码问题"
date: 2013-05-31 21:33
comments: true
categories: openstack 
---

{% img right /images/main/OpenStack.jpeg 200 100 'openstack' 'openstack' %}

openstack的sql connection 需要配置成utf-8才不会中文乱码

```
sql_connection = mysql://nova:xxx@qa-mysql1:3306/nova?charset='utf-8'
```

但是如果之前就没有设置utf-8，数据库又不能随便清空，想要更新openstack数据库某个字段，就不能直接采用utf-8的数据库连接，否则会出现插入到数据库中的是中文，但是openstack中显示的是乱码。更坏的是如果插入的是全角中文字符，openstack就会出错。 

<!--more-->

那么如何更新这种情况下的数据库呢？

答案很简单，就是采用和openstack一样的数据库连接方式，openstack使用的是 sqlalchemy
我写一个一个简单的数据库连接如下

```
#!/usr/bin/python 
# coding=utf-8

from sqlalchemy import create_engine
from sqlalchemy import MetaData
from sqlalchemy.sql import select
from sqlalchemy.sql import update

engine = create_engine('mysql://nova:password@qa-mysql1:3306/nova',convert_unicode=True)

metadata=MetaData()
meta=metadata
meta.reflect(bind=engine)
conn=engine.connect()
secgroup=meta.tables['security_groups']

u=update(secgroup).where(secgroup.c.id==60).values(name='中文')
result=conn.execute(u)
```


这里需要注意的是在`create_engine`的时候，需要添加`convert_unicode=True`,如果不添加，会出如下解码错误的错误

```
    query = query % db.literal(args)
  File "/usr/share/pyshared/MySQLdb/connections.py", line 264, in literal
    return self.escape(o, self.encoders)
  File "/usr/share/pyshared/MySQLdb/connections.py", line 202, in unicode_literal
    return db.literal(u.encode(unicode_literal.charset))
UnicodeEncodeError: 'latin-1' codec can't encode characters in position 0-1: ordinal not in range(256)
```



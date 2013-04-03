---
layout: post
title: "openstack 无法删除虚拟机 一例"
date: 2013-04-03 13:18
comments: true
catagories: "cloud"
---


运行`nova delete ceph01`删除虚拟机ceph01,发现无法删除虚拟机，

1. controller节点
===============

* 查看虚拟机状态
```
# nova show ceph01
+-------------------------------------+----------------------------------------------------------+
| Property                            | Value                                                    |
+-------------------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                   |
| OS-EXT-SRV-ATTR:host                | compute2                                                 |
| OS-EXT-SRV-ATTR:hypervisor_hostname | compute2                                                 |
| OS-EXT-SRV-ATTR:instance_name       | instance-00000114                                        |
| OS-EXT-STS:power_state              | 1                                                        |
| OS-EXT-STS:task_state               | deleting                                                 |
| OS-EXT-STS:vm_state                 | active                                                   |

```

<!-- more -->
* 查看日志发现无任何报错

2. compute节点
===============


* 虚拟机所在的物理机上` /var/log/nova/nova-compute.log`有如下错误


```
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp     return self.api.client.post(url, body=body)
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp   File "/usr/lib/python2.7/dist-packages/cinderclient/client.py", line 141, in post
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp     return self._cs_request(url, 'POST', **kwargs)
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp   File "/usr/lib/python2.7/dist-packages/cinderclient/client.py", line 126, in _cs_request
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp     **kwargs)
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp   File "/usr/lib/python2.7/dist-packages/cinderclient/client.py", line 109, in request
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp     raise exceptions.from_response(resp, body)
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp ClientException: The server has either erred or is incapable of performing the requested operation. (HTTP 500) (Request-ID: req-a9122ed1-9198-4464-8da9-7458fb581379)
2013-04-03 13:04:17 45732 TRACE nova.openstack.common.rpc.amqp
2013-04-03 13:04:17 DEBUG nova.utils [req-195b0553-4ff9-4cd5-9e06-63324e8865da 77e43059b67e4c139cb6a9cbb08ef20d f7de462059fb419f90f37006dd4e27e9] Got semaphore "92e08fe5-89a3-494c-85b9-96072bcb051b" for method "do_terminate_instance"... inner /usr/lib/python2.7/dist-packages/nova/utils.py:765

```

* 使用 ` virsh list --all` 发现虚拟机应经不存在了
```
# virsh list 
 Id    Name                           State
----------------------------------------------------
 108   instance-000000f4              running
 110   instance-000000f5              running
 122   instance-0000010c              running
 123   instance-0000010d              running
 124   instance-00000110              running
 128   instance-0000011b              running

```

3. 解决方法
之前部署过多controller节点，然后将其关闭了，可能导致删除的时候，compute节点消息没有传递回来，具体机制还不明朗。
目前先清空数据库中相应的条目

```
root@controller3:/var/log/cinder# mysql -uroot -p
Enter password: 
mysql> use nova;

mysql> update  instances set deleted=1 where display_name='ceph01';
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> quit
Bye

```


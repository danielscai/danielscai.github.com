---
layout: post
title: "quantum l3 agent 多外部网问题(quantum muliple l3 agent issue)"
date: 2013-05-18 12:19
comments: true
categories: openstack
---

{% img right /images/main/OpenStack.jpeg 200 100 'openstack' 'openstack' %}
quantum l3 agent 多外部网问题

问题描述


在 network 节点上`/var/log/quantum/l3-agent.log` 查看到如下log，

<!--more-->
主要是`The 'gateway_external_network_id' must be configured if Quantum has more than one external network.`这句.

```
Stdout: '8: br-ex: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN \\    link/ether fe:4d:08:7b:3d:4a brd ff:ff:ff:ff:ff:ff\n'
Stderr: ''
2013-05-17 14:43:30    DEBUG [quantum.openstack.common.rpc.amqp] Making synchronous call on q-plugin ...
2013-05-17 14:43:30    DEBUG [quantum.openstack.common.rpc.amqp] MSG_ID is 642c0e87c8f44f179d45485039d5d879
2013-05-17 14:43:30    DEBUG [quantum.openstack.common.rpc.amqp] UNIQUE_ID is d9ae832d62134946ac39a3537088e8a4.
2013-05-17 14:43:30    ERROR [quantum.agent.l3_agent] Failed synchronizing routers
Traceback (most recent call last):
  File "/usr/lib/python2.7/dist-packages/quantum/agent/l3_agent.py", line 639, in _sync_routers_task
    self._process_routers(routers, all_routers=True)
  File "/usr/lib/python2.7/dist-packages/quantum/agent/l3_agent.py", line 589, in _process_routers
    target_ex_net_id = self._fetch_external_net_id()
  File "/usr/lib/python2.7/dist-packages/quantum/agent/l3_agent.py", line 223, in _fetch_external_net_id
    raise Exception(msg)
Exception: The 'gateway_external_network_id' must be configured if Quantum has more than one external network.
2013-05-17 14:43:32    DEBUG [quantum.openstack.common.rpc.amqp] Making synchronous call on q-plugin ...

```

在folsom中，这个参数是一定要配置的，否则无法工作，但是在grizzly版中， 这个是可以不配置的，如果只有一个外部网络，默认使用这个网络。


**查看有哪些外部网络**

使用`quantum net-list`无法直接查看到哪些是外部网络，需要加上过滤器

```
# quantum net-list  -- --router:external=True
+--------------------------------------+------------+-------------------------------------------------------+
| id                                   | name       | subnets                                               |
+--------------------------------------+------------+-------------------------------------------------------+
| 2465e4cd-cd02-4a52-bafa-6bf060bee6ae | net_extral |                                                       |
| c30aa6ff-3169-442d-8da1-baf54272b4c8 | public_net | 03828046-9965-4659-88f3-9380114268fc 192.168.0.0/24   |
|                                      |            | 3d255d61-561c-42aa-b0cf-e90b05f895ec 192.168.22.0/24  |
|                                      |            | 7c363a4d-4ce8-4160-a92b-80753ef0d25b 192.168.100.0/24 |
+--------------------------------------+------------+-------------------------------------------------------+

```
 

在network节点上查看这个qrouter的namespace 会报如下错误

```
# ip netns exec qrouter-8afc5f1e-ee99-4af5-a7f8-ec2edefebddd ip a
seting the network namespace failed: Invalid argument
```



解决方法

删除多个外部网路，只留一个外部网络，
或者在network 节点配置l3 agent 加上`gateway_external_network_id`配置


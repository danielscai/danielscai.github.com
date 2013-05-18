---
layout: post
title: "quantum 网络创建使用指南(quantum create network guide)"
date: 2013-05-18 12:24
comments: true
categories: openstack
---

{% img right /images/main/OpenStack.jpeg 200 100 'openstack' 'openstack' %}

quantum 网络创建使用指南

1. 创建网络
---

    quantum net-create network1

<!--more-->

2. 创建子网
---

```
quantum subnet-create network1 192.168.12.0/24
Created a new subnet:
+------------------+----------------------------------------------------+
| Field            | Value                                              |
+------------------+----------------------------------------------------+
| allocation_pools | {"start": "192.168.12.2", "end": "192.168.12.254"} |
| cidr             | 192.168.12.0/24                                    |
| dns_nameservers  |                                                    |
| enable_dhcp      | True                                               |
| gateway_ip       | 192.168.12.1                                       |
| host_routes      |                                                    |
| id               | 32fe8d37-5c35-453e-b7d4-a1c98ac7bf97               |
| ip_version       | 4                                                  |
| name             |                                                    |
| network_id       | 54f37003-7f4e-47f4-a865-c928de1ae9e3               |
| tenant_id        | 180deec2c933457cb149ef5cf38322f8                   |
+------------------+----------------------------------------------------+
```

3. 创建外部网络
---

3.1 创建外部网络目的
---

作为floating ip 池

3.2 创建外部网络注意点
---

创建外部网络子网的时候，注意network 节点上的br-ex网桥一定是要存在的

`eth2`接口的配置在`ubuntu`里可以参考如下的配置

    auto eth2
    iface eth2 inet manual
    up ifconfig $IFACE 0.0.0.0 up
    up ip link set $IFACE promisc on
    down ip link set $IFACE promisc off
    down ifconfig $IFACE down
这个子网能工作的前提条件必须满足,不然即使分配了`floating ip`也是无法工作的

* 这个子网一定一个是物理上和你`network`节点连接的,这里是用的eth2接口
* `allocation-pool`中的ip都是可用的
* 不需要dhcp
* `network`节点上`openvswitch`网桥`br-ex`必须存在
* `br-ex`上添加`eth2`接口，命令为`ovs-vsctl add-port br-ex eth2`

下面是建好的ovs网桥

```
bridge name bridge id  STP enabled interfaces
br-ex  0000.180373d2869d no  eth2
br-int  0000.823fbdfff741 no  qr-520f18b3-85
      tap7415fb87-d7
      tape9e5dc79-c8
      tapfec2a3b4-4c
br-tun  0000.5ef67744584e no
virbr0  8000.000000000000 yes

```

3.3 创建外部网络

```
# quantum net-create ext_net --router:external=True
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | 45217501-16c6-49a9-946d-6d44ef2b0256 |
| name                      | ext_net                              |
| provider:network_type     | gre                                  |
| provider:physical_network |                                      |
| provider:segmentation_id  | 2                                    |
| router:external           | True                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | 180deec2c933457cb149ef5cf38322f8     |
+---------------------------+--------------------------------------+

```

3.3. 创建外部网络子网
---

这里我们创建一个`10.1.3.0/24`的子网，假设你的的`eth2`接口是接在`10.1.3.0/24`网段的，
分配地址从140到160,就是在参数中指定`--allocation-pool start=10.1.3.10,end=10.1.3.70`, 这里的dhcp是需要关闭的

```
# quantum subnet-create --allocation-pool start=10.1.3.10,end=10.1.3.70 --gateway 10.1.3.1 ext_net 10.1.3.0/24 --enable_dhcp=False
Created a new subnet:
+------------------+--------------------------------------------+
| Field            | Value                                      |
+------------------+--------------------------------------------+
| allocation_pools | {"start": "10.1.3.10", "end": "10.1.3.70"} |
| cidr             | 10.1.3.0/24                                |
| dns_nameservers  |                                            |
| enable_dhcp      | False                                      |
| gateway_ip       | 10.1.3.1                                   |
| host_routes      |                                            |
| id               | e51a2803-64cf-4d8d-9637-9ae38ad4d76c       |
| ip_version       | 4                                          |
| name             |                                            |
| network_id       | 45217501-16c6-49a9-946d-6d44ef2b0256       |
| tenant_id        | 180deec2c933457cb149ef5cf38322f8           |
+------------------+--------------------------------------------+
```

3.4 查看创建好的网络
---

```
# quantum net-list 
+--------------------------------------+----------+------------------------------------------------------+
| id                                   | name     | subnets                                              |
+--------------------------------------+----------+------------------------------------------------------+
| 45217501-16c6-49a9-946d-6d44ef2b0256 | ext_net  | e51a2803-64cf-4d8d-9637-9ae38ad4d76c 10.1.3.0/24     |
| 54f37003-7f4e-47f4-a865-c928de1ae9e3 | network1 | 32fe8d37-5c35-453e-b7d4-a1c98ac7bf97 192.168.12.0/24 |
+--------------------------------------+----------+------------------------------------------------------+
```

4. 创建路由
---

**4.1 创建路由**

```
# quantum router-create default-router
Created a new router:
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| admin_state_up        | True                                 |
| external_gateway_info |                                      |
| id                    | 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a |
| name                  | default-router                       |
| status                | ACTIVE                               |
| tenant_id             | 180deec2c933457cb149ef5cf38322f8     |
+-----------------------+--------------------------------------+
```

**4.2 将路由添加到一个l3 agent上**

查看有哪些l3 agent 

···
# quantum agent-list 
+--------------------------------------+--------------------+----------+-------+----------------+
| id                                   | agent_type         | host     | alive | admin_state_up |
+--------------------------------------+--------------------+----------+-------+----------------+
| 20fe37a3-9146-4744-9f4a-67db973b70e2 | Open vSwitch agent | network2 | :-)   | True           |
| 40dabe35-dfc1-4723-9ab7-5b4dfb51108b | L3 agent           | network2 | :-)   | True           |
| 6452b796-e7f6-4057-96df-4d320a0ceb84 | DHCP agent         | network1 | :-)   | True           |
| 6dcd47a8-f56e-46db-aec1-5c2c18f0c321 | Open vSwitch agent | network1 | :-)   | True           |
| 715ce6bc-7cb5-4e0b-9de9-3b6a8750cf59 | Open vSwitch agent | compute1 | :-)   | True           |
| a39da87a-16ea-4924-8aff-d9e468e68f13 | DHCP agent         | network2 | :-)   | True           |
| ccf26580-ba40-4148-9a73-c0cf67713a73 | L3 agent           | network1 | :-)   | True           |
+--------------------------------------+--------------------+----------+-------+----------------+
···

**4.3 将路由添加到network1的l3 agent上**

命令如下
      
    quantum l3-agent-router-add $l3_agent_ID router_id

···
# quantum l3-agent-router-add  ccf26580-ba40-4148-9a73-c0cf67713a73 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a 
Added router 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a to L3 agent
···

**4.4 设置路由网关**

命令如下`quantum router-gateway-set $router_id $ext_net_id`

```
# quantum router-gateway-set 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a  45217501-16c6-49a9-946d-6d44ef2b0256 
Set gateway for router 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a
```

查看路由
```
# quantum router-list 
+--------------------------------------+----------------+--------------------------------------------------------+
| id                                   | name           | external_gateway_info                                  |
+--------------------------------------+----------------+--------------------------------------------------------+
| 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a | default-router | {"network_id": "45217501-16c6-49a9-946d-6d44ef2b0256"} |
+--------------------------------------+----------------+--------------------------------------------------------+
```


**4.5 将路由添加到子网 **

或者说将子网添加到路由，主要目的就是让某个子网的路由设置成我们的router,这个router 的网关已经被设置为外部网络，这样我们的子网就可以上网了
命令如下`quantum router-interface-add $router_id $subnet_id`

```
# quantum router-interface-add 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a  32fe8d37-5c35-453e-b7d4-a1c98ac7bf97
Added interface to router 93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a
```


network节点上应该会出现一个qrouter的namespace,如果启用了namespace的话（默认启动）

```
# ip netns
qrouter-93a70ed7-ac17-41d3-9d92-4d8bfb0ee56a
```

**4.6 创建一个floating ip **

```
# quantum floatingip-create ext_net 
Created a new floatingip:
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| fixed_ip_address    |                                      |
| floating_ip_address | 10.1.3.11                            |
| floating_network_id | 45217501-16c6-49a9-946d-6d44ef2b0256 |
| id                  | 9461b279-c5c7-4db0-a332-a3466c616861 |
| port_id             |                                      |
| router_id           |                                      |
| tenant_id           | 180deec2c933457cb149ef5cf38322f8     |
+---------------------+--------------------------------------+
```


至此，网络相关资源全部创建完成

下面我们来验证网络是否工作正常


5. 启动虚拟机，分配floating ip 
---

** 创建port **

···
# quantum port-create --fixed-ip subnet_id=32fe8d37-5c35-453e-b7d4-a1c98ac7bf97 54f37003-7f4e-47f4-a865-c928de1ae9e3 
Created a new port:
+----------------------+-------------------------------------------------------------------------------------+
| Field                | Value                                                                               |
+----------------------+-------------------------------------------------------------------------------------+
| admin_state_up       | True                                                                                |
| binding:capabilities | {"port_filter": false}                                                              |
| binding:vif_type     | ovs                                                                                 |
| device_id            |                                                                                     |
| device_owner         |                                                                                     |
| fixed_ips            | {"subnet_id": "32fe8d37-5c35-453e-b7d4-a1c98ac7bf97", "ip_address": "192.168.12.2"} |
| id                   | 69802c75-20a9-4533-9b26-8975a39f9a5d                                                |
| mac_address          | fa:16:3e:af:35:30                                                                   |
| name                 |                                                                                     |
| network_id           | 54f37003-7f4e-47f4-a865-c928de1ae9e3                                                |
| status               | DOWN                                                                                |
| tenant_id            | 180deec2c933457cb149ef5cf38322f8                                                    |
+----------------------+-------------------------------------------------------------------------------------+
···

**创建 volume**
···
# cinder create --image-id 6b01928b-056f-4d2e-bb33-988a4853d7f9 --display-name dcai-precise 10 
+---------------------+--------------------------------------+
|       Property      |                Value                 |
+---------------------+--------------------------------------+
|     attachments     |                  []                  |
|  availability_zone  |                 nova                 |
|       bootable      |                false                 |
|      created_at     |      2013-05-17T08:32:55.108870      |
| display_description |                 None                 |
|     display_name    |             dcai-precise             |
|          id         | 949077a3-67d9-478c-82b1-a57ad6780e97 |
|       image_id      | 6b01928b-056f-4d2e-bb33-988a4853d7f9 |
|       metadata      |                  {}                  |
|         size        |                  10                  |
|     snapshot_id     |                 None                 |
|     source_volid    |                 None                 |
|        status       |               creating               |
|     volume_type     |                 None                 |
+---------------------+--------------------------------------+
···

**启动虚拟机**

```
# nova boot --flavor 1 --image 6b01928b-056f-4d2e-bb33-988a4853d7f9 --block_device_mapping vda=949077a3-67d9-478c-82b1-a57ad6780e97:::0 --nic port-id=69802c75-20a9-4533-9b26-8975a39f9a5d dcai-m1
+-------------------------------------+--------------------------------------+
| Property                            | Value                                |
+-------------------------------------+--------------------------------------+
| OS-EXT-STS:task_state               | scheduling                           |
| image                               | precise                              |
| OS-EXT-STS:vm_state                 | building                             |
| OS-EXT-SRV-ATTR:instance_name       | instance-0000007b                    |
| flavor                              | m1.tiny                              |
| id                                  | 9e6c3ed0-7e7a-4fdf-ae66-9b9eb99163b7 |
| security_groups                     | [{u'name': u'default'}]              |
| user_id                             | d61a9823e00f417b90b842dd8b5f7a94     |
| OS-DCF:diskConfig                   | MANUAL                               |
| accessIPv4                          |                                      |
| accessIPv6                          |                                      |
| progress                            | 0                                    |
| OS-EXT-STS:power_state              | 0                                    |
| OS-EXT-AZ:availability_zone         | None                                 |
| config_drive                        |                                      |
| status                              | BUILD                                |
| updated                             | 2013-05-17T08:35:01Z                 |
| hostId                              |                                      |
| OS-EXT-SRV-ATTR:host                | None                                 |
| key_name                            | None                                 |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                 |
| name                                | dcai-m1                              |
| adminPass                           | XH2o4LJda7CE                         |
| tenant_id                           | 180deec2c933457cb149ef5cf38322f8     |
| created                             | 2013-05-17T08:35:01Z                 |
| metadata                            | {}                                   |
+-------------------------------------+--------------------------------------+
```

进入虚拟机，查看一切是否正常

---
layout: post
title: "openstack 启动时指定使用子网
(openstack boot with subnet port)"
date: 2013-04-25 12:36
comments: true
categories: openstack
---

{% img right /images/main/OpenStack.jpeg 250 150 'openstack' 'openstack' %}

openstack 启动时指定使用子网

在调用`nova boot`命令时，默认是创建所有网络的所有接口

那么有没有办法指定使用某个网络，或者使用某个子网呢

答案是肯定的

<!-- more --> 

首先我们先创建一个网络，在这个网络下创建2个子网用来测试

1. 创建网络和子网
---

**创建网络**

创建一个名称为`demo-net2`的网络

    root@qa-controller1:~# quantum net-create demo-net2
    Created a new network:
    +---------------------------+--------------------------------------+
    | Field                     | Value                                |
    +---------------------------+--------------------------------------+
    | admin_state_up            | True                                 |
    | id                        | f9d3bd8e-377b-4f21-bfc6-64ae4257e44d |
    | name                      | demo-net2                            |
    | provider:network_type     | gre                                  |
    | provider:physical_network |                                      |
    | provider:segmentation_id  | 4                                    |
    | router:external           | False                                |
    | shared                    | False                                |
    | status                    | ACTIVE                               |
    | subnets                   |                                      |
    | tenant_id                 | 82da519b676d400ab24e9ee38d138c3c     |
    +---------------------------+--------------------------------------+
    
**创建两个子网**

在`demo-net2`下面创建两个子网 `20.20.1.0/24`和 `20.20.2.0/24`

    root@qa-controller1:~# quantum subnet-create demo-net2 20.20.1.0/24 
    Created a new subnet:
    +------------------+----------------------------------------------+
    | Field            | Value                                        |
    +------------------+----------------------------------------------+
    | allocation_pools | {"start": "20.20.1.2", "end": "20.20.1.254"} |
    | cidr             | 20.20.1.0/24                                 |
    | dns_nameservers  |                                              |
    | enable_dhcp      | True                                         |
    | gateway_ip       | 20.20.1.1                                    |
    | host_routes      |                                              |
    | id               | 3a6c08b6-cb0e-4949-9e3f-dae76fc98741         |
    | ip_version       | 4                                            |
    | name             |                                              |
    | network_id       | f9d3bd8e-377b-4f21-bfc6-64ae4257e44d         |
    | tenant_id        | 82da519b676d400ab24e9ee38d138c3c             |
    +------------------+----------------------------------------------+
    
    root@qa-controller1:~# quantum subnet-create demo-net2 20.20.2.0/24 
    Created a new subnet:
    +------------------+----------------------------------------------+
    | Field            | Value                                        |
    +------------------+----------------------------------------------+
    | allocation_pools | {"start": "20.20.2.2", "end": "20.20.2.254"} |
    | cidr             | 20.20.2.0/24                                 |
    | dns_nameservers  |                                              |
    | enable_dhcp      | True                                         |
    | gateway_ip       | 20.20.2.1                                    |
    | host_routes      |                                              |
    | id               | dfe2a150-5a87-4256-80c7-fd88b9dae113         |
    | ip_version       | 4                                            |
    | name             |                                              |
    | network_id       | f9d3bd8e-377b-4f21-bfc6-64ae4257e44d         |
    | tenant_id        | 82da519b676d400ab24e9ee38d138c3c             |
    +------------------+----------------------------------------------+
    
2. 启动虚拟机，应用一个网络的所有子网
---

这个比较简单，在nova boot的时候加上`--nic net-id=net-uuid`就可以了
下面主要介绍一下如何在启动虚拟机时，应用一个子网


3. 启动虚拟机，应用一个子网
---

首先需要从这个子网中获得一个port-id ,比如我们想使用demo-net2中的 `20.20.2.0/24` 子网地址

运行下面的命令，获得一个port-id (53770585-e26b-488e-916c-cede86087084 ) 

    root@qa-controller1:~# quantum port-create --fixed-ip subnet_id=dfe2a150-5a87-4256-80c7-fd88b9dae113 f9d3bd8e-377b-4f21-bfc6-64ae4257e44d 
    Created a new port:
    +----------------------+----------------------------------------------------------------------------------+
    | Field                | Value                                                                            |
    +----------------------+----------------------------------------------------------------------------------+
    | admin_state_up       | True                                                                             |
    | binding:capabilities | {"port_filter": false}                                                           |
    | binding:vif_type     | ovs                                                                              |
    | device_id            |                                                                                  |
    | device_owner         |                                                                                  |
    | fixed_ips            | {"subnet_id": "dfe2a150-5a87-4256-80c7-fd88b9dae113", "ip_address": "20.20.2.3"} |
    | id                   | 53770585-e26b-488e-916c-cede86087084                                             |
    | mac_address          | fa:16:3e:7a:e8:36                                                                |
    | name                 |                                                                                  |
    | network_id           | f9d3bd8e-377b-4f21-bfc6-64ae4257e44d                                             |
    | status               | DOWN                                                                             |
    | tenant_id            | 82da519b676d400ab24e9ee38d138c3c                                                 |
    +----------------------+----------------------------------------------------------------------------------+

启动虚拟机，指定该 port-id 

    # nova boot --flavor 1 --block_device_mapping vda=21f4dfc5-9752-4f5e-8133-0540a1dc3eb5:::0 --nic port-id=53770585-e26b-488e-916c-cede86087084 m1
    +-------------------------------------+-------------------------------------------------+
    | Property                            | Value                                           |
    +-------------------------------------+-------------------------------------------------+
    | OS-EXT-STS:task_state               | scheduling                                      |
    | image                               | Attempt to boot from volume - no image supplied |
    | OS-EXT-STS:vm_state                 | building                                        |
    | OS-EXT-SRV-ATTR:instance_name       | instance-00000072                               |
    | flavor                              | m1.tiny                                         |
    | id                                  | e1122239-9dbb-40db-870e-2810ed07a1fd            |
    | security_groups                     | [{u'name': u'default'}]                         |
    | user_id                             | 6f8e4ee659b642ce898aaefe201d0eaa                |
    | OS-DCF:diskConfig                   | MANUAL                                          |
    | accessIPv4                          |                                                 |
    | accessIPv6                          |                                                 |
    | progress                            | 0                                               |
    | OS-EXT-STS:power_state              | 0                                               |
    | OS-EXT-AZ:availability_zone         | None                                            |
    | config_drive                        |                                                 |
    | status                              | BUILD                                           |
    | updated                             | 2013-04-25T02:21:26Z                            |
    | hostId                              |                                                 |
    | OS-EXT-SRV-ATTR:host                | None                                            |
    | key_name                            | None                                            |
    | OS-EXT-SRV-ATTR:hypervisor_hostname | None                                            |
    | name                                | m1                                              |
    | adminPass                           | 6UFo62FEcMUQ                                    |
    | tenant_id                           | 82da519b676d400ab24e9ee38d138c3c                |
    | created                             | 2013-04-25T02:21:26Z                            |
    | metadata                            | {}                                              |
    +-------------------------------------+-------------------------------------------------+

删除虚拟机后，该port也会被删除




---
layout: post
title: "openstack迁移环境引起的permission denied 错误"
date: 2013-05-31 21:49
comments: true
categories: openstack
---

{% img right /images/main/OpenStack.jpeg 200 100 'openstack' 'openstack' %}

openstack 迁移错误

openstack 迁移到新的环境。

在新的环境中启动一台虚拟机，发现一直是error状态

查看虚拟机状态提示`ComputeHostNotFound_Remote`和`Compute host 1 could not be found. `

但是其实并没有compute 1 节点。 
<!--more-->

如下


```
root@controller2:~# nova show d4e4059c-7306-4773-8e01-360a236fd44b
+-----------------------------+--------------------------------------------------------------------------------------------------------------+
| Property                    | Value                                                                                                        |
+-----------------------------+--------------------------------------------------------------------------------------------------------------+
| status                      | ERROR                                                                                                        |
| updated                     | 2013-05-29T12:21:36Z                                                                                         |
| OS-EXT-STS:task_state       | None                                                                                                         |
| key_name                    | None                                                                                                         |
| image                       | precise (6b01928b-056f-4d2e-bb33-988a4853d7f9)                                                               |
| hostId                      | fc6914630cd8a0c64cfaf744b2e85aff0b7c58d16565ff9d5a48ff12                                                     |
| OS-EXT-STS:vm_state         | error                                                                                                        |
| flavor                      | m1.tiny (1)                                                                                                  |
| id                          | d4e4059c-7306-4773-8e01-360a236fd44b                                                                         |
| security_groups             | [{u'name': u'default'}]                                                                                      |
| user_id                     | 05e8636f0cb94171bba93f5a1a73c927                                                                             |
| name                        | api_test                                                                                                     |
| created                     | 2013-05-29T12:20:57Z                                                                                         |
| fault                       | {u'message': u'ComputeHostNotFound_Remote', u'code': 404, u'details': u'Compute host 1 could not be found.   |
|                             | Traceback (most recent call last):                                                                           |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/openstack/common/rpc/amqp.py", line 430, in _process_data      |
|                             |     rval = self.proxy.dispatch(ctxt, version, method, **args)                                                |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/openstack/common/rpc/dispatcher.py", line 133, in dispatch     |
|                             |     return getattr(proxyobj, method)(ctxt, **kwargs)                                                         |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/conductor/manager.py", line 352, in compute_node_update        |
|                             |     prune_stats)                                                                                             |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/db/api.py", line 200, in compute_node_update                   |
|                             |     return IMPL.compute_node_update(context, compute_id, values, prune_stats)                                |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/db/sqlalchemy/api.py", line 96, in wrapper                     |
|                             |     return f(*args, **kwargs)                                                                                |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/db/sqlalchemy/api.py", line 545, in compute_node_update        |
|                             |     compute_ref = _compute_node_get(context, compute_id, session=session)                                    |
|                             |                                                                                                              |
|                             |   File "/usr/lib/python2.7/dist-packages/nova/db/sqlalchemy/api.py", line 455, in _compute_node_get          |
|                             |     raise exception.ComputeHostNotFound(host=compute_id)                                                     |
|                             |                                                                                                              |
|                             | ComputeHostNotFound: Compute host 1 could not be found.                                                      |
|                             | ', u'created': u'2013-05-29T12:21:36Z'}                                                                      |
| OS-DCF:diskConfig           | MANUAL                                                                                                       |
| metadata                    | {u'image_id': u'6b01928b-056f-4d2e-bb33-988a4853d7f9', u'os_name': u'ubuntu', u'My Server Name': u'Apache1'} |
| accessIPv4                  |                                                                                                              |
| accessIPv6                  |                                                                                                              |
| tenant_id                   | a32328ccd3f641c882910a32cdff91fc                                                                             |
| OS-EXT-STS:power_state      | 0                                                                                                            |
| OS-EXT-AZ:availability_zone | nova                                                                                                         |
| config_drive                |                                                                                                              |
+-----------------------------+--------------------------------------------------------------------------------------------------------------+

```


查看 ` nova scheduler ` 错误

```

2013-05-29 20:21:34.481 ERROR nova.scheduler.filter_scheduler [req-3b009407-bcbb-46a3-8ae9-173574cf7854 05e8636f0cb94171bba93f5a1a73c927 a32328ccd3f641c882910a32cdff91fc] [instance: d4e4059c-7306-4773-8e01-360a236fd44b] Error from last host: compute3 (node compute3): [u'Traceback (most recent call last):\n', u'  File "/usr/lib/python2.7/dist-packages/nova/compute/manager.py", line 834, in _run_instance\n    set_access_ip=set_access_ip)\n', u'  File "/usr/lib/python2.7/dist-packages/nova/compute/manager.py", line 1093, in _spawn\n    LOG.exception(_(\'Instance failed to spawn\'), instance=instance)\n', u'  File "/usr/lib/python2.7/contextlib.py", line 24, in __exit__\n    self.gen.next()\n', u'  File "/usr/lib/python2.7/dist-packages/nova/compute/manager.py", line 1089, in _spawn\n    block_device_info)\n', u'  File "/usr/lib/python2.7/dist-packages/nova/virt/libvirt/driver.py", line 1520, in spawn\n    block_device_info)\n', u'  File "/usr/lib/python2.7/dist-packages/nova/virt/libvirt/driver.py", line 2433, in _create_domain_and_network\n    self.firewall_driver.setup_basic_filtering(instance, network_info)\n', u'  File "/usr/lib/python2.7/dist-packages/nova/virt/libvirt/firewall.py", line 288, in setup_basic_filtering\n    self.refresh_provider_fw_rules()\n', u'  File "/usr/lib/python2.7/dist-packages/nova/virt/firewall.py", line 476, in refresh_provider_fw_rules\n    self._do_refresh_provider_fw_rules()\n', u'  File "/usr/lib/python2.7/dist-packages/nova/openstack/common/lockutils.py", line 221, in inner\n    with lock:\n', u'  File "/usr/lib/python2.7/dist-packages/nova/openstack/common/lockutils.py", line 75, in __enter__\n    self.lockfile = open(self.fname, \'w\')\n', u"IOError: [Errno 13] Permission denied: '/run/lock/nova/nova-iptables'\n"]
```


解决方案 

/var/lock/nova 的权限是 nova:root , 应该是nova:nova

```
chown -R nova:nova /var/lock/nova
```

---
layout: post
title: "quantum Exceeded maximim amount of fixed ips per port 错误 "
date: 2013-05-31 21:39
comments: true
categories: openstack
---

{% img right /images/main/OpenStack.jpeg 200 100 'openstack' 'openstack' %}

openstack 运行过程中突然出现这个错误

在 network节点上`/var/log/quantum/dhcp-agent.log`

出现如下错误日志

提示` Exceeded maximim amount of fixed ips per port.`

应该是跟quota有关的。

<!--more-->

```
2013-05-22 16:31:10    ERROR [quantum.agent.dhcp_agent] Unable to enable dhcp.
Traceback (most recent call last):
  File "/usr/lib/python2.7/dist-packages/quantum/agent/dhcp_agent.py", line 129, in call_driver
    getattr(driver, action)()
  File "/usr/lib/python2.7/dist-packages/quantum/agent/linux/dhcp.py", line 117, in enable
    reuse_existing=True)
  File "/usr/lib/python2.7/dist-packages/quantum/agent/dhcp_agent.py", line 528, in setup
    port = self.plugin.get_dhcp_port(network.id, device_id)
  File "/usr/lib/python2.7/dist-packages/quantum/agent/dhcp_agent.py", line 377, in get_dhcp_port
    topic=self.topic))
  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/proxy.py", line 80, in call
    return rpc.call(context, self._get_topic(topic), msg, timeout)
  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/__init__.py", line 140, in call
    return _get_impl().call(CONF, context, topic, msg, timeout)
  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/impl_kombu.py", line 798, in call
    rpc_amqp.get_connection_pool(conf, Connection))
  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/amqp.py", line 613, in call
    rv = list(rv)
  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/amqp.py", line 562, in __iter__
    raise result
RemoteError: Remote error: InvalidInput Invalid input for operation: Exceeded maximim amount of fixed ips per port.
[u'Traceback (most recent call last):\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/amqp.py", line 430, in _process_data\n    rval = self.proxy.dispatch(ctxt, version, method, **args)\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/common/rpc.py", line 43, in dispatch\n    quantum_ctxt, version, method, **kwargs)\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/openstack/common/rpc/dispatcher.py", line 133, in dispatch\n    return getattr(proxyobj, method)(ctxt, **kwargs)\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/db/dhcp_rpc_base.py", line 104, in get_dhcp_port\n    dict(port=port))\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/plugins/openvswitch/ovs_quantum_plugin.py", line 612, in update_port\n    context, id, port)\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/db/db_base_plugin_v2.py", line 1341, in update_port\n    p[\'fixed_ips\'])\n', u'  File "/usr/lib/python2.7/dist-packages/quantum/db/db_base_plugin_v2.py", line 657, in _update_ips_for_port\n    raise q_exc.InvalidInput(error_message=msg)\n', u'InvalidInput: Invalid input for operation: Exceeded maximim amount of fixed ips per port.\n'].
```


解决方法

在quantum server上设置 `/etc/quantum/quantum.conf `


```
# Maximum number of fixed ips per port
max_fixed_ips_per_port = 1000
```



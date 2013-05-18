---
layout: post
title: "quantum agent 名称为 localhost 问题(quantum agent with name localhost issue)"
date: 2013-05-18 11:24
comments: true
categories: openstack
---

{% img right /images/main/OpenStack.jpeg 200 100 'openstack' 'openstack' %}
quantum agent 名称为 localhost 问题

两个环境中发现其中一个环境中quantum的agent 名称变成了ip6-localhost

运行`quantum agent-list`发现agent 名称都是localhost 或者是ip6-localhost 

如下

<!--more-->

```
# quantum agent-list
+--------------------------------------+--------------------+---------------+-------+----------------+
| id                                   | agent_type         | host          | alive | admin_state_up |
+--------------------------------------+--------------------+---------------+-------+----------------+
| 468a79a0-0135-4e0e-9594-b19c7e5a7b2a | L3 agent           | ip6-localhost | :-)   | True           |
| 8c484bb0-15e3-4ed0-b74f-b0a9d3916a32 | Open vSwitch agent | localhost     | :-)   | True           |
| a758b8ce-6d59-4e5e-b3a9-9f1d1079be6b | Open vSwitch agent | ip6-localhost | :-)   | True           |
| b334115b-a2fd-49a1-891c-dd492dda084e | DHCP agent         | ip6-localhost | :-)   | True           |
```


**问题原因**

agent所在节点的`/etc/hosts`文件中配置了 `127.0.0.1 localhost compute1`这样的记录，导致agent 所在host反解自身名称的时候识别为localhost 

**修复问题**

1.将所有agent服务停止
---


```
cd /etc/init.d/
for i in quantum-* ; do service $i stop; done
```

2.清空所有agent 
---

    for i in ` quantum agent-list | grep -v "\---"  | grep -v "^\ id" | awk '{print $2}' `;do quantum agent-delete $i ; done



3. 重新启动agent 
---

```
cd /etc/init.d/
for i in quantum-* ; do service $i restart; done
```

检查

```
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
```

---
layout: post
title: "openstack grizzly quantum-metadata-agent debug"
date: 2013-04-12 09:36
comments: true
categories: openstack
---

{% img right /images/main/grizzly.jpg 250 150 'puppet' 'puppet' %}
一个错误的配置导致 quantum-metadata-agent 无法启动，也没有相关log

在`/var/log/quantum/metadata-agent.log` 中没有发现任何log


在`/var/log/syslog`中只有如下一条简单的错误日志
<!-- more -->

```
Apr 11 15:24:47 network1 kernel: [ 4511.402916] init: quantum-metadata-agent main process (26599) terminated with status 1
```


查看启动脚本 `/etc/init.d/quantum-metadata-agent` 发现是一个upstart job
 
查看 `/etc/init/quantum-metadata-agent.conf` 

```
# cat /etc/init/quantum-metadata-agent.conf 
description "Quantum metadata plugin agent"
author "Yolanda Robla <yolanda.robla@canonical.com>"

start on runlevel [2345]
stop on runlevel [016]

chdir /var/run

pre-start script
mkdir -p /var/run/quantum
chown quantum:root /var/run/quantum
end script

exec start-stop-daemon --start --chuid quantum --exec /usr/bin/quantum-metadata-agent -- \
            --config-file=/etc/quantum/quantum.conf --config-file=/etc/quantum/metadata_agent.ini \
            --log-file=/var/log/quantum/metadata-agent.log
```


手动运行这条命令,找到python的报错信息，提示配置文件错误 ` Section must be started before assignment`


```
# /usr/bin/quantum-metadata-agent --config-file=/etc/quantum/quantum.conf --config-file=/etc/quantum/metadata_agent.ini --log-file=/var/log/quantum/metadata-agent.log
Traceback (most recent call last):
  File "/usr/bin/quantum-metadata-agent", line 20, in <module>
    main()
  File "/usr/lib/python2.7/dist-packages/quantum/agent/metadata/agent.py", line 223, in main
    cfg.CONF(project='quantum')
  File "/usr/lib/python2.7/dist-packages/oslo/config/cfg.py", line 1180, in __call__
    self._parse_config_files()
  File "/usr/lib/python2.7/dist-packages/oslo/config/cfg.py", line 1651, in _parse_config_files
    raise ConfigFileParseError(pe.filename, str(pe))
oslo.config.cfg.ConfigFileParseError: Failed to parse /etc/quantum/metadata_agent.ini: at /etc/quantum/metadata_agent.ini:3, Section must be started before assignment: None

```

导致这个错误的原因很简单 ` /etc/quantum/metadata_agent.ini ` 开头的`[DEFAULT]`被漏掉了

```
# cat /etc/quantum/metadata_agent.ini 
[DEFAULT]
# The Quantum user information for accessing the Quantum API.
auth_url = http://controller1:35357/v2.0
auth_region = RegionOne
admin_tenant_name = service
admin_user = quantum
admin_password = service_pass

# IP address used by Nova metadata server
nova_metadata_ip = 172.16.136.211

# TCP Port used by Nova metadata server
nova_metadata_port = 8775

metadata_proxy_shared_secret = helloOpenStack

```


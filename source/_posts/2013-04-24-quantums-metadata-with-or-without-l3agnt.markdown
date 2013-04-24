---
layout: post
title: "quantum's metadata with or without l3agnt "
date: 2013-04-24 21:00
comments: true
categories: openstack
---

{% img right /images/main/grizzly.jpg 250 150 'puppet' 'puppet' %}

openstack grizzly 中的 quantum 的`metadata`可以和`l3 agent`一起工作   

也可以不依赖`l3 agent `独立工作

或者l3 agent 配合 dhcp agent ，不需要metadata agent 

<!-- more -->

1.当和l3 agent一起工作时
---

网络包直接发送到 169.254.169.254的包被发送到默认网关,`l3 agent`会为没一个路由启动一个`metadata proxy ` 将流量转到 nova metadata server 上

2. 当没有l3 agent 时
---
没有`l3 agent`的时候，需要在`dhcp agent`的配置文件中加入 

    enable_isolated_metadata_proxy= True

这样 dhcp agent 会往所有主机中加入一条静态路由 `169.254.0.0/16`
dhcp agent 会为没一个网络启动一个`metadata proxy` ，将流量转到nova metadata server 上

3. 有l3 agent ，却不想部署metadata agent 
---

依然是有办法部署这一套系统的，只需要添加如下配置文件到`dhcp agent`的配置文件

    enable_metadata_network=True
    
    
以下是原文

>    Quantum's metadata solution for Grizzly can run either with or without the l3 agent.

>    When running within the l3 agent, packets directed to 169.254.169.254 are sent to the default gateway; the l3 agent will spawn a metadata proxy for each router; the metadata proxy forwards them to the metadata agent using a datagram socket, and finally the agent reaches the Nova metadata server.
    
>    Without the l3 agent, the 'isolated' mode can be enabled for the metadata access service. This is achieved by setting the flag enable_isolated_metadata_proxy to True in the dhcp_agent configuration file. When the isolated proxy is enabled, the dhcp agent will send an additional static route to each VM. This static route will have the dhcp agent as next hop and 169.254.0.0/16 as destination CIDR; the dhcp agent will spawn a metadata proxy for each network. Once the packet reaches the proxy, the procedure works as above. This should also explain why the metadata agent does not depend on the l3 agent.
    
>    If you are deploying the l3 agent, but do not want to deploy the metadata agent on the same host, the 'metadata access network' can be considered. This option is enabled by setting enable_metadata_network on the dhcp agent configuration file. When enabled, quantum networks whose cidr is included in 169.254.0.0/16 will be regarded as 'metadata networks', and will spawn a metadata proxy. The user can then connect such network to any logical router through the quantum API; thus granting metadata access to all the networks connected to such router.
    
>    I think the documentation for quantum metadata has not yet been merged in the admin guide.
>    I hope this clarifies the matter a little... although this thread has gone a little bit off-topic. Can you consider submitting one or more questions to ask.openstack.org?


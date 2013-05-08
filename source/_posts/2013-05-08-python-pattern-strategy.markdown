---
layout: post
title: "python 设计模式之 策略模式 "
date: 2013-05-08 23:27
comments: true
categories: coding
---

{% img right /images/main/python-logo.gif 250 150 'python' 'python' %}

策略模式的主体思想是将变与不变的部分分离。

概括的说，不变的部分我们可以写成类生成的一个实例，或者模块。将变的部分写成一个类，然后每次根据条件的不同生成多个实例，再将这个变的实例通过参数的方式传入到不变的实例或者模块中。

<!--more-->




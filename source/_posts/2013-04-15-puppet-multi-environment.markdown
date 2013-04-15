---
layout: post
title: "git版本控制下，配置puppet多环境"
date: 2013-04-15 09:08
comments: true
categories: coding
---


本文档主要讨论在实际环境下的puppet多环境配置，并使用git作为版本控制。

配置多环境主要目的在于

- 隔离生产环境和测试开发环境
- 提供多个开发者独立环境


0. 配置puppet多环境
==================
详细配置puppet多环境请参考puppet官方文档
http://docs.puppetlabs.com/guides/environment.html
我们这里件4个环境production ,test,dev_a和dev_b
`/etc/puppet/puppet.conf` 配置如下
```
[production]
modulepath = $confdir/env/production/modules
manifest = $confdir/env/production/manifests/site.pp
[test]
modulepath = $confdir/env/test/modules
manifest = $confdir/env/test/manifests/site.pp
[dev_a]
modulepath = $confdir/env/dev_a/modules
manifest = $confdir/env/dev_a/manifests/site.pp
[dev_b]
modulepath = $confdir/env/dev_b/modules
manifest = $confdir/env/dev_b/manifests/site.pp
```
<!-- more -->

1.建立git仓库
==================
1.1 建立两个git仓库
----------------------------
一个用来管理puppet中的modules，一个用来管理除了modules以外的所有内容
我们将仓库称为 puppet 和modules , 其中modules 作为puppet 的一个子模块，子模块可以参考http://gitbook.liuhui998.com/5_10.html

仓库可以建立在github或者任意你喜欢的git托管仓库上，这里我们建两个本地仓库作为演示,
注意，如果是不是使用的本地仓库，请在commit后`git push`到server上
```
cd /tmp
mkdir puppet modules
```
1.2 初始化两个仓库
----------------------------
初始化仓库，并将modules作为子模块添加到 puppet仓库中,此模块作为默认环境，也就是production环境的modules文件

```
cd modules ; git init ; cd .. 
cd puppet ; git init
git submodule add ../modules env/production/modules
```

1.3添加排除文件
----------------------------
因为使用了modules子模块，因此我们需要将env/*/modules排除在puppet仓库的控制外，这样多个开发者就可以在各自的环境下开发而不影响当前puppet仓库
但是我们的env/production/modules是一个子模块，仍然是需要控制的，因此我们将`.gitignore`设置为
```
env/*/modules
!env/production/modules
```

2. 管理puppet配置文件
==================
2.1 将现有环境中的puppet配置文件copy到puppet仓库下
----------------------------
```
cp -r /etc/puppet/* ./
git add . 
git commit -m 'init' 
```

2.2 管理modules
----------------------------
```
cd ../modules/
cp -r /etc/puppet/env/production/modules/* ./
git add .
git commit -m 'add modules'
```

3. 更新子模块
==================
```
cd ..
git submodule update
```

4.多开发者共同开发
==================
开发者在各自的分支中开发，将测试好的代码`merge`到主干分支
下面演示一个开发者(dev_a)建立自己的开发环境

4.1 拷贝`manifest`
----------------------------
```
cd /tmp/puppet/env/dev_a
cp -r ../production/manifests ./
```

4.2 建立modules分支
----------------------------
```
git clone ../production/modules modules
cd modules/
git checkout -b dev_a
```

4.3 将开发版本合并到主干
----------------------------
开发者将完成开发任务并测试后将代码合并到主干分支，如果有测试团队，将代码先合并到测试团队分支是一个好主意，这里就直接合并到主干分支(`master`分支)上
```
touch 1
git add 1
git commit -m 'complete task 1 '
```
到目前为止开发都在dev_a分支上，当完成测试后，合并到主干
```
git checkout master
git merge dev_a --no-ff
git push origin master
```
对于为何使用`--no-ff`参数，可以参考http://www.ruanyifeng.com/blog/2012/07/git.html

在prodution环境签出最新代码
```
cd ../../production/modules/
git pull origin master
```


5. 部署新的puppet环境
将仓库中的puppet配置部署到新的环境中
在新的环境中安装好`puppet`，将配置文件备份到其他目录，签出puppet代码和其子模块的代码
以`ubuntu 12.04 precise`为例,将 `/tmp/puppet`替换成你真实的git仓库地址

```
wget http://apt.puppetlabs.com/puppetlabs-release-precise.deb
dpkg -i puppetlabs-release-precise.deb
apt-get update
apt-get upgrade

apt-get install puppetmaster 
mv /etc/puppet /etc/puppet.old 
cd /etc/

git clone /tmp/puppet puppet
git submodule init
git submodule update
```


---
layout: post
title: "ubuntu 12.04 部署octopress"
date: 2013-04-20 13:01
comments: true
categories: os
---
"deploy octopress in ubuntu 12.04"

1.安装ruby
=======================
1.1安装1.9.3
-------------------------------------
```
apt-get -y install ruby1.9.3
```

<!-- more -->
1.2.更新ruby 
-------------------------------------

```
sudo update-alternatives --install /usr/bin/ruby ruby /usr/bin/ruby1.9.3 400 \
         --slave   /usr/share/man/man1/ruby.1.gz ruby.1.gz \
                        /usr/share/man/man1/ruby1.9.3.1.gz \
        --slave   /usr/bin/ri ri /usr/bin/ri1.9.3 \
        --slave   /usr/bin/irb irb /usr/bin/irb1.9.3 \
        --slave   /usr/bin/rdoc rdoc /usr/bin/rdoc1.9.3
```
更改ruby使用1.9.3版本
```
sudo update-alternatives --config ruby
sudo update-alternatives --config gem
```

2.部署octopress 环境
====================

2.1.clone octopress
-------------------------------
```
git clone https://github.com/imathis/octopress.git
```


2.2 修改Gemfile的source，使用淘宝的gem源
-------------------------------
```
source "http://ruby.taobao.org"
```

2.3 安装gem包
-------------------------------
```
gem install bundler
bundle install 
rake install 
```

3.部署到github pages
====================
3.1 建立github 仓库
-------------------------------
新建一个以username.github.com  格式的github仓库，username是你的用户名

3.2 部署
-------------------------------
部署到github pages ,按提示填写github仓库
```
rake setup_github_pages
```

```
rake generate
rake deploy
```

3.3 自定义
-------------------------------
自定义设定可以在 `_config.yaml`中设置，包括 `github`,`google analytics`,`google plus` 等


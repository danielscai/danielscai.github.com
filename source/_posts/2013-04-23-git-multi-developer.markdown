---
layout: post
title: "git 多开发者协作模式"
date: 2013-04-23 14:35
comments: true
categories: coding
---


在多开发者共同开发一个项目时，git 的多分支可以帮助多个开发者在各自环境下开发而不影响到团队其他人员。

下面是多个开发者在各自环境下工作时用到的一些命令

1. 创建分支
===
每个开发者需要有一个自己的分支，比如一个新的开发人员加入项目中，可能需要用到下面的命令创建分支

**克隆新代码**

    git clone git_repo_path project_dir 
    
<!-- more --> 

**创建一个本地分支**
我们创建一个名叫 `dcai`的分支

    cd project_dir
    git checkout -b dcai
    
**将本地分支提交到远程分支**

    git push origin dcai 
    
至此，一个新的分支在远程和本地就创建好了

**列出所有分支**

    # git branch -a 
    * dcai
      master
      remotes/origin/HEAD -> origin/master
      remotes/origin/dcai
      remotes/origin/master

2.在分支上工作
===

在本地`dcai`分支上做出改动并测试通过后，将本次改动`commit`到远程`dcai`分支上 

**查看本次修改**

我在文件底部测试性的加了一个空行

    # git diff 
    diff --git a/manage.py b/manage.py
    index f087353..e2d695f 100755
    --- a/manage.py
    +++ b/manage.py
    @@ -14,3 +14,4 @@ if __name__ == "__main__":
         manager.run()
         #app.run(host='0.0.0.0',port=8088)
     
    +

**提交改动，并push到git远程仓库中**

    git add manage.py
    git commit -m 'test, add a new line'
    git push origin dcai
    

3. 合并到主干
===

当你确定自己分支上的代码已经没有问题的时候，可以将代码合并到主干，就是`master`分支上 ，你需要做如下几个步骤

**切换到主干分支**

    git checkout master
    
**合并你的分支到主干**

    git merge dcai --no-ff
    
合并的时候如果有冲突，需要自己手动解决冲突。
最后将合并结果`push`到远程git仓库中

    `git push origin master`
    
**最后不要忘记切换到自己的分支上，继续工作**

    `git checkout dcai`
    
    
    

    

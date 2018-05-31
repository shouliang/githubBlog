title: 七.自定义Git
date: 2014-06-29 19:45:23
tags: [Git]
---

### 1.自定义Git
---
在安装Git一节中，我们已经配置了user.name和user.email，实际上，Git还有很多可配置项。

比如，让Git显示颜色，会让命令输出看起来更醒目：
```
$ git config --global color.ui true
```
这样，Git会适当地显示不同的颜色，比如git status命令：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkgitColor01.png)

文件名就会标上颜色。
### 2.忽略特殊文件
---
有些时候我们需要把一些文件例如：保存了数据库密码的配置文件放在Git目录下，但又不提交，那么需要我们在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。

　　不需要从头写.gitignore文件，GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：https://github.com/github/gitignore

忽略文件的原则是：

* 忽略操作系统自动生成的文件，比如缩略图等；
* 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件；
* 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

### 3.配置别名
---
配置别名其实就是把命令重新设置简单些，方便输入，例如：如果输入git st就表示git status。
```
$ git config --global alias.st status
```
现在都用co表示checkout，ci表示commit，br表示branch：
```
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
```
--global参数是全局参数，也就是这些命令在这台电脑的所有Git仓库下都有用。

### 4.配置文件
---
配置Git的时候，加上--global是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。

配置文件放哪了？每个仓库的Git配置文件都放在.git/config文件中：

$ cat .git/config 
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
    ignorecase = true
    precomposeunicode = true
[remote "origin"]
    url = git@github.com:michaelliao/learngit.git
    fetch = +refs/heads/\*:refs/remotes/origin/\*
[branch "master"]
    remote = origin
    merge = refs/heads/master
[alias]
    last = log -1
   
别名就在[alias]后面，要删除别名，直接把对应的行删掉即可。

而当前用户的Git配置文件放在用户主目录下的一个隐藏文件.gitconfig中：

$ cat .gitconfig
[alias]
    co = checkout
    ci = commit
    br = branch
    st = status
[user]
    name = Your Name
    email = your@email.com
   
配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置。

### 5.搭建Git服务器
---
搭建Git服务器需要准备一台运行Linux的机器，强烈推荐用Ubuntu或Debian，这样，通过几条简单的apt命令就可以完成安装。

假设你已经有sudo权限的用户账号，下面，正式开始安装。

__第一步__，安装git：
``` 
$ sudo apt-get install git
``` 
__第二步__，创建一个git用户，用来运行git服务：
``` 
$ sudo adduser git
``` 
__第三步__，创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。

__第四步__，初始化Git仓库：

先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令：
``` 
$ sudo git init --bare sample.git
``` 
　　Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：
``` 
$ sudo chown -R git:git sample.git
``` 
__第五步__，禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：
``` 
git:x:1001:1001:,,,:/home/git:/bin/bash
``` 
改为：
``` 
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
``` 
这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

__第六步__，克隆远程仓库：

现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：
``` 
$ git clone git@server:/srv/sample.git
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
``` 
### 6.管理公钥和权限
---
#### 管理公钥
　　如果团队很小，把每个人的公钥收集起来放到服务器的/home/git/.ssh/authorized_keys文件里就是可行的。如果团队有几百号人，就没法这么玩了，这时，可以用Gitosis来管理公钥。

#### 管理权限
　　有很多不但视源代码如生命，而且视员工为窃贼的公司，会在版本控制系统里设置一套完善的权限控制，每个人是否有读写权限会精确到每个分支甚至每个目录下。因为Git是为Linux源代码托管而开发的，所以Git也继承了开源社区的精神，不支持权限控制。不过，因为Git支持钩子（hook），所以，可以在服务器端编写一系列脚本来控制提交等操作，达到权限控制的目的。Gitolite就是这个工具。

这里我们也不介绍Gitolite了，不要把有限的生命浪费到权限斗争中。

主要两点：

* 要方便管理公钥，用Gitosis；
* 要像SVN那样变态地控制权限，用Gitolite。

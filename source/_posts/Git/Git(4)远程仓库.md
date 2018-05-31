title: 四.远程仓库
date: 2014-06-23 17:45:23
tags: [Git]
---

### 1.远程仓库简介
---
到目前为止，我们已经掌握了如何在Git仓库里对一个文件进行时光穿梭，你再也不用担心文件备份或者丢失的问题了。

　　开始介绍Git的远程仓库功能。我们现在借用GitHub神奇的网站，这个网站就是提供Git仓库托管服务的，所以只要注册一个GitHub账号，就可以免费获得Git远程仓库。

　　由于你的本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以，需要一点设置：

　　第1步：创建SSH Key。在当前目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

$ ssh-keygen -t rsa -C "youremail@example.com"
　　你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

　　第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：

　　然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容：
　　![](http://7xq1il.com1.z0.glb.clouddn.com/mkgitsshkey.png)

### 2.GitHub创建新仓库
---
GitHub创建一个Git仓库,并且本地仓库与此仓库进行远程同步，此仓库既可以作为备份，又可以让其他人通过该仓库来协作。

　　首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库：
　　
　　![](http://7xq1il.com1.z0.glb.clouddn.com/mkgithub1.png)
　　
　在Repository name填入learngit，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库。

　　目前，在GitHub上的这个learngit仓库还是空的，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。

　　现在，我们根据GitHub的提示，在本地的learngit仓库下运行命令：
```
$ git remote add origin git@github.com:onlyone/learngit.git
```
　　请千万注意，把上面的onlyone替换成你自己的GitHub账户名，否则，你在本地关联的就是我的远程库，关联没有问题，但是你以后推送是推不上去的，因为你的SSH Key公钥不在我的账户列表中。

　　添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。

　　注意：要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git。　
　　
### 3.本地库推送到远程库
---
本地库的所有内容推送到远程库上：

$ git push -u origin master
　　把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。

　　由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

　　推送成功，github内容与当前目录中readme.txt一致：
　　
　　![](http://7xq1il.com1.z0.glb.clouddn.com/mkgithubremote.png)
　　
从现在起，只要本地作了提交，就可以通过命令：
```
$ git push origin master
```
　　把本地master分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库！

　　注意：要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git；关联后，使用命令git push -u origin master第一次推送master分支的所有内容；此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；
　　
### 4.SSH警告
---
当你第一次使用Git的clone或者push命令连接GitHub时，会得到一个警告：
```
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
```
　　这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入yes回车即可。

　　Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：
```
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
```
　　这个警告只会出现一次，后面的操作就不会有任何警告了。

　　如果你实在担心有人冒充GitHub服务器，输入yes前可以对照GitHub的RSA Key的指纹信息是否与SSH连接给出的一致。

### 5.从远程库克隆
---
从远程库克隆,就需要我们先创建远程库，在github创建一个新的gitskills仓库，我们勾选Initialize this repository with a README，这样GitHub会自动为我们创建一个README.md文件。创建完毕后，可以看到README.md文件：

　　现在，远程库已经准备好了，下一步是用命令git clone克隆一个本地库：
```
$ git clone git@github.com:michaelliao/gitskills.git
 
$ cd gitskills
$ ls
README.md
```
　　注意把Git库的地址换成你自己的，然后进入gitskills目录看看，已经有README.md文件了。

　　你也许还注意到，GitHub给出的地址不止一个，还可以用github.com/onlyone/gitskills.git这样的地址。实际上，Git支持多种协议，默认的git://使用ssh，但也可以使用https等其他协议。

　　注意：要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。

　　Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。


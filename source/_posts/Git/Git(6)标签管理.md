title: 六.标签管理
date: 2014-06-27 17:45:23
tags: [Git]
---

### 1.标签的简介
---
本节主要记录的Git标签的作用、标签的多种创建方式，以及标签的删除，与推送，和使用GitHub的Fork参与别人的项目。

__标签的作用__
　　发布版本时，通常先在版本库中打一个标签，这样，就唯一确定了打标签时刻的版本。无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针（跟分支很像，但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的。
　　
### 2.创建标签
---
在Git中打标签非常简单，首先，切换到需要打标签的分支上：

$ git branch
\*dev
  master
$ git checkout master
Switched to branch 'master'
然后，敲命令git tag <name>就可以打一个新标签：
```
$ git tag v1.0
```
　　默认标签是打在最新提交的commit上的。还可以对历史提交打上标签，只要找到历史提交的commit id，然后打上就可以了，例如要对add merge这次提交打标签，它对应的commit id是6224937，输入命令：
```
$ git tag v0.9 6224937
```
　　还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
```
$ git tag -a v0.1 -m "version 0.1 released" 3628164
```
　　用命令git show 可以看到说明文字：
```
$ git show v0.1
tag v0.1
Tagger: hubwiz <hubwiz@163.com>
Date:   Mon Aug 26 07:28:11 2015 +0800
version 0.1 released
```
　　签名采用PGP签名，因此，必须首先安装gpg（GnuPG），如果没有找到gpg，或者没有gpg密钥对，就会报错：
```
gpg: signing failed: secret key not available
error: gpg failed to sign the data
error: unable to sign the tag
```
　　如果报错，请参考GnuPG帮助文档配置Key。

* 命令git tag <name>用于新建一个标签，默认为HEAD，也可以指定一个commit id；
* git tag -a <tagname> -m "blablabla..."可以指定标签信息；
* git tag -s <tagname> -m "blablabla..."可以用PGP签名标签；
* 命令git tag可以查看所有标签。

### 3.操作标签
---
如果标签打错了，也可以删除：
```
$ git tag -d v0.1
Deleted tag 'v0.1' (was e078af9)
```
因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

如果要推送某个标签到远程，使用命令it push origin <tagname>:
```
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0
```
或者，一次性推送全部尚未推送到远程的本地标签：
```
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 554 bytes, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:michaelliao/learngit.git
* [new tag]         v0.2 -> v0.2
* [new tag]         v0.9 -> v0.9
```
如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：
```
$ git tag -d v0.9
Deleted tag 'v0.9' (was 6224937)
```
然后，从远程删除。删除命令也是push，但是格式如下：
```
$ git push origin :refs/tags/v0.9
To git@github.com:michaelliao/learngit.git
 - [deleted]         v0.9
 ```
要看看是否真的从远程库删除了标签，可以登陆GitHub查看。

本节知识点主要学习点：

* git push origin <tagname>可以推送一个本地标签；
* git push origin --tags可以推送全部未推送过的本地标签；
* git tag -d <tagname>可以删除一个本地标签；
* git push origin :refs/tags/<tagname>可以删除一个远程标签。

### 4.GitHub的使用
---
GitHub不仅是免费的远程仓库，个人的开源项目，可以放到GitHub上，而且GitHub还是一个开源协作社区，通过GitHub，既可以让别人参与你的开源项目，也可以参与别人的开源项目。

　　在GitHub上，利用Git极其强大的克隆和分支功能，人们可以自由参与各种开源项目。比如人气极高的bootstrap项目，这是一个非常强大的CSS框架，在它的项目主页，点“Fork”就在自己的账号下克隆了一个bootstrap仓库，然后，从自己的账号下clone。一定要从自己的账号下clone仓库，这样你才能推送修改。如果从bootstrap的作者的仓库地址git@github.com:twbs/bootstrap.git克隆，因为没有权限，你将不能推送修改。

　　Bootstrap的官方仓库twbs/bootstrap、你在GitHub上克隆的仓库my/bootstrap，以及你自己克隆到本地电脑的仓库，他们的关系就像下图显示的那样：
　　
![](http://7xq1il.com1.z0.glb.clouddn.com/mkbootstrap.png)

如果你希望bootstrap的官方库能接受你的修改，你就可以在GitHub上发起一个pull request。

本节主要内容：

* 在GitHub上，可以任意Fork开源仓库；
* 自己拥有Fork后的仓库的读写权限；
* 可以推送pull request给官方仓库来贡献代码。
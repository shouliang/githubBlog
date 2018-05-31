title: 三.操作Git
date: 2014-06-20 14:45:23
tags: [Git]
---

### 1.status命令与diff命令
---
前面我们已经成功地添加并提交了一个readme.txt文件，修改readme.txt如下：
```
echo "Git is a distributed version control system. " > readme.txt
echo "Git is free software." >> readme.txt
```
　　运行git status命令看看结果：
```
$ git status
...
no changes added to commit (use "git add" and/or "git commit -a")
```
　　git status命令可以让我们时刻掌握仓库当前的状态，上面显示，readme.txt被修改过了，但还没有准备提交的修改。

　　git diff这个命令看看：
```
$ git diff readme.txt 
...
-Git is version control system.                                             
+Git is a distributed version control system.
 Git is free software
```
　　git diff顾名思义就是查看difference，显示的格式正是Unix通用的diff格式，可以从上面的命令输出看到，我们在第一行添加了一个“distributed”单词。

　　readme.txt作了什么修改后，再把它提交到仓库，提交修改和提交新文件是一样的两步，git add和git commit：
```
$ git add readme.txt
$ git commit -m "add distributed"
```
注意
* 要随时掌握工作区的状态，使用git status命令。
* 如果git status告诉你有文件被修改过，用git diff可以查看修改内容。

### 2.版本回退
---
现在，再练习一次，修改readme.txt文件如下：
```
echo "Git is a distributed version control system." > readme.txt
echo "Git is free software distributed under the GPL." >> readme.txt
```
　　我们再次提交一次readme.txt
```
$ git add readme.txt
$ git commit -m "append GPL"
```
　　我们现在已经提交多次文件，想看看有那些？版本控制系统肯定有某个命令可以告诉我们历史记录，在Git中，我们用git log命令查看：
```
$ git log
commit 3628164fb26d48395383f8f31179f24e0882e1e0
Date:   Tue Oct 25 15:11:49 2015 +0000
    append GPL
commit ea34578d5496d7dd233c827ed32a8cd576c5ee85
Date:   Tue Oct 25 14:53:12 2015 +0000
    add distributed
commit cb926e7ea50ad11b8f9e909c05226233bf755030
Date:   Mon Oct 24 17:51:55 2015 +0000
    wrote a readme file
```
　　git log命令显示从最近到最远的提交日志，我们可以看到3次提交，最近的一次是append GPL，上一次是add distributed，最早的一次是wrote a readme file。commit 36281**2e1e0是commit id（版本号）。如果嫌输出信息太多，可以使用$ git log --pretty=oneline，此时你看到的一大串类似3628164...882e1e0的是commit id（版本号）。

　　每提交一个新版本，实际上Git就会把它们自动串成一条时间线。现在准备把readme.txt回退到上一个版本，也就是“add distributed”的那个版本，怎么做呢？

　　首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交3628164...882e1e0（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。

　　现在，我们要把当前版本“append GPL”回退到上一个版本“add distributed”，就可以使用git reset命令：
```
$ git reset --hard HEAD^
HEAD is now at ea34578 add distributed
```
### 3.重新恢复到新版本
---
接着上节版本回退，还可以继续回退到上一个版本wrote a readme file，不过我们现在看看版本库的状态git log：
```
$ git log
```
　　最新的那个版本append GPL已经看不到了！好比你从21世纪坐时光穿梭机来到了19世纪，想再回去已经回不去了，肿么办？

　　只要之前的命令行结果还在，就可以找到那个append GPL的commit id是3628164...，于是就可以指定回到未来的某个版本：
```
$ git reset --hard 3628164
HEAD is now at 3628164 append GPL
```
　　版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了。

　　可以查看readme.txt的内容$ cat readme.txt.

　　Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向append GPL：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkback01.jpg)
改为指向add distributed：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkback02.jpg)
### 4.git reflog命令
---
现在，你回退到了某个版本，当想恢复到新版本怎么办？找不到新版本的commit id怎么办？

　　在Git中可以放心下。当你用$ git reset --hard HEAD^回退到add distributed版本时，再想恢复到append GPL，就必须找到append GPL的commit id。Git提供了一个命令git reflog用来记录你的每一次命令：
```
$ git reflog
ea34578 HEAD@{0}: reset: moving to HEAD^
3628164 HEAD@{1}: commit: append GPL
ea34578 HEAD@{2}: commit: add distributed
cb926e7 HEAD@{3}: commit (initial): wrote a readme file
```
　　这样可以看到，第二行显示append GPL的commit id是3628164，这样我们就可以重新找到了。

　　注意，我们从这两节中可以了解到：

HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git reset --hard commit_id。
穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。
要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。

### 5.工作区
---
__工作区__：就是你在电脑里能看到的目录,learngit文件夹就是一个工作区，比如我们环境中当前的目录。

![](http://7xq1il.com1.z0.glb.clouddn.com/mkworkArea.png)

### 6.暂存区
---
__版本库:__工作区有一个隐藏目录.git 这个不算工作区，而是Git的版本库。

__暂存区:__英文叫stage,或index。一般存放在git 目录下的index文件(.git/index)中，所以我们把暂存区时也叫作索引(index).

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。

![](http://7xq1il.com1.z0.glb.clouddn.com/mkstage.jpg)

### 7.暂存区实践
---
现在我们对readme.txt做个修改，比如追加一行内容：
```
echo "Git has a mutable index called stage." >> readme.txt
```
然后，在工作区新增一个LICENSE文本文件
```
echo "LICENSE is a new file." > LICENSE
```
用git status查看一下状态，Git显示结果，readme.txt被修改了，而LICENSE还从来没有被添加过，所以它的状态是Untracked。

现在，使用两次命令git add，把readme.txt和LICENSE都添加后，用git status再查看一下，通过图可以理解为

![](http://7xq1il.com1.z0.glb.clouddn.com/mkstorageArea01.jpg)
所以，git add命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行git commit就可以一次性把暂存区的所有修改提交到分支。
```
$ git commit -m "understand how stage works"
```
一旦提交后，如果你又没有对工作区做任何修改，用git status查看下，没有任何内容，现在版本库变成了这样，暂存区就没有任何内容了：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkstorageArea02.jpg)

### 8.管理修改
---
Git与其他版本控制系统相比，Git跟踪并管理的是修改，而非文件。

　　为什么说Git管理的是修改，而不是文件。接下来做测试，我们对readme.txt修改，追加一行内容：
```
echo "Git tracks changes." >> readme.txt 
```
　　然后通过git add添加
```
$ git add readme.txt
$ git status
```
　　接下来，我们再次修改readme.txt内容，把最后一行内容修改为：
```
Git tracks changes of files.
```
　　git commit提交
```
$ git commit -m "git tracks changes"
[master d4f25b6] git tracks changes
 1 file changed, 1 insertion(+)
 ```
　　通过git status查看每次操作的状态，可以看出第二次修改 -> git commit，没有被提交。

　　Git管理的是修改，当你用git add命令后，在工作区的第一次修改被放入暂存区，准备提交，但是，在工作区的第二次修改并没有放入暂存区，所以，git commit只负责把暂存区的修改提交了，也就是第一次的修改被提交了，第二次的修改不会被提交。

Git是如何跟踪修改的，每次修改，如果不add到暂存区，那就不会加入到commit中。

### 9.撤销修改
---
如果你在readme.txt中加入了一行文件，又感觉不好，你可以删除新加的，恢复到原来的。

Git会告诉你，git checkout -- file可以丢弃工作区的修改：
 ```
$ git checkout -- readme.txt
 ```
命令git checkout -- readme.txt意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

* 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
* 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
总之，就是让这个文件回到最近一次git commit或git add时的状态。

现在，看看readme.txt的文件内容：
 ```
$ cat readme.txt
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
 ```
现在来看几种情况，如何撤销修改

1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步：

第一步用命令git reset HEAD file，就回到了1；
第二步按1操作。
3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

### 10.删除文件
---
在Git中，删除也是一个修改操作，先添加一个新文件test.txt到Git并且提交：
 ```
$ git add test.txt
$ git commit -m "add test.txt"
[master 94cdc44] add test.txt
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
 ```
一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用rm命令删了：
 ```
$ rm test.txt
 ```
这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了：
 ```
$ git status
...
no changes added to commit (use "git add" and/or "git commit -a")
 ```
现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit：
 ```
$ git rm test.txt
rm 'test.txt'
$ git commit -m "remove test.txt"
 ```
现在，文件就从版本库中被删除了。

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：
 ```
$ git checkout -- test.txt
 ```
git checkout其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

　　命令git rm用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

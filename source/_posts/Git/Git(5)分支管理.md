title: 五.分支管理
date: 2014-06-24 17:45:23
tags: [Git]
---

### 1.分支管理图文详解一
---
在版本回退里，你已经知道，每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。

　　一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点：
　　
![](http://7xq1il.com1.z0.glb.clouddn.com/mkbranch01.png)

每次提交，master分支都会向前移动一步，这样，随着你不断提交，master分支的线也越来越长，当我们创建新的分支，例如dev时，Git新建了一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev上：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkbranch02.png)

你看，Git创建一个分支很快，因为除了增加一个dev指针，改改HEAD的指向，工作区的文件都没有任何变化！

### 2.分支管理图文详解二
---
接着分支管理图文详解一，现在对工作区的修改和提交就是针对dev分支了，比如新提交一次后，dev指针往前移动一步，而master指针不变：
![](http://7xq1il.com1.z0.glb.clouddn.com/mkbranch03.png)

假如我们在dev上的工作完成了，就可以把dev合并到master上。Git怎么合并呢？最简单的方法，就是直接把master指向dev的当前提交，就完成了合并：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkbranch04.png)

所以Git合并分支也很快！就改改指针，工作区内容也不变！

合并完分支后，甚至可以删除dev分支。删除dev分支就是把dev指针给删掉，删掉后，我们就剩下了一条master分支：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkbranch05.png)

### 3.创建分支
---
首先，我们创建dev分支，然后切换到dev分支：
```
$ git checkout -b dev
Switched to a new branch 'dev'
```
git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：

$ git branch dev
$ git checkout dev
Switched to branch 'dev'

然后，用git branch命令查看当前分支：

$ git branch
\*dev
  master

git branch命令会列出所有分支，当前分支前面会标一个*号。

然后，我们就可以在dev分支上正常提交，比如对readme.txt做个修改，加上一行：

Creating a new branch is quick.

然后提交：

$ git add readme.txt 
$ git commit -m "branch test"
[dev fec145a] branch test
1 file changed, 1 insertion(+)

现在，dev分支的工作完成，我们就可以切换回master分支：

```
$ git checkout master
Switched to branch 'master'
```

切换回master分支后，再查看一个readme.txt文件，刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有变：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkdev.png)

### 4.合并分支
---
我们把前面dev分支的工作成果合并到master分支上：

$ git merge dev
Updating d17efd8..fec145a
Fast-forward
 readme.txt |    1 +
 1 file changed, 1 insertion(+)

　　git merge命令用于合并指定分支到当前分支。合并后，再查看readme.txt的内容，就可以看到，和dev分支的最新提交是完全一样的。

　　注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

　　当然，也不是每次合并都能Fast-forward，我们后面会将其他方式的合并。

### 5.删除分支
---
合并完成后，就可以放心地删除dev分支了：
```
$ git branch -d dev
Deleted branch dev (was 5659891).
```
删除后，查看branch，就只剩下master分支了：
```
$ git branch
* master
```
因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

前面所讲知识：汇总下这些使用命令：

查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>

创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>

### 6.产生冲突
---
当我们进行合并分支往往会产生冲突。

在准备新的feature1分支，继续我们的新分支开发
```
$ git checkout -b feature1
```
修改readme.txt最后一行，改为Creating a new branch is quick AND simple.

在feature1分支上提交
```
$ git add readme.txt 
$ git commit -m "AND simple"
```
切换到master分支
```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
```
Git还会自动提示我们当前master分支比远程的master分支要超前1个提交。

在master分支上把readme.txt文件的最后一行改为Creating a new branch is quick & simple.提交：
```
$ git add readme.txt 
$ git commit -m "& simple"
```
### 7.解决冲突
---
在上节中master分支和feature1分支各自都分别有新的提交，变成了这样：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkresolve01.png)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，执行git merge feature1,在看readme.txt：
```
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```
我们把冲突的内容修改为Creating a new branch is quick and simple.，提交：
```
$ git add readme.txt 
$ git commit -m "conflict fixed"
```
现在，master分支和feature1分支变成了下图所示：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkresolve02.png)

用带参数的git log也可以看到分支的合并情况：
```
$ git log --graph --pretty=oneline --abbrev-commit
```
最后，删除feature1分支：
```
$ git branch -d feature1
Deleted branch feature1.
```
冲突解决，最后，删除feature1分支 git branch -d feature1。当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

用git log --graph命令可以看到分支合并图。

### 8.Bug分支
---
如果你有一个bug任务，你想创建一个分支issue-101来修复它，但是你当前正在dev上进行的工作还没有完成而不能提交，bug需要现在修复，所以现在你需要暂停dev上工作，Git提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：$ git stash。

　　假定需要在master分支上修复，就从master创建临时分支：
```
$ git checkout master
Switched to branch 'master'
$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```
　　现在修复bug，需要把“Git is free software ...”改为“Git is a free software ...”，然后提交：
```
$ git add readme.txt 
$ git commit -m "fix bug 101"
```
　　修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：
```
$ git checkout master
...
$ git merge --no-ff -m "merged bug fix 101" issue-101
...
$ git branch -d issue-101
Deleted branch issue-101 (...).
```
　　Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

* 一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；
* 另一种方式是用git stash pop，恢复的同时把stash内容也删了：

### 9.Feature分支
---
　在软件开发中，总会添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

　　现在新功能开发代号为Vulcan:
```
$ git checkout -b feature-vulcan
Switched to a new branch 'feature-vulcan'
```
　　开发完毕，添加并提交：
```
$ git add vulcan.py
$ git commit -m "add feature vulcan"
```
切回dev，准备合并：
```
$ git checkout dev
```
　　一切顺利的话，feature分支和bug分支是类似的，合并，然后删除。

　　由于种种原因，此功能又不需要了，现在这个分支需要就地销毁：
```
$ git branch -d feature-vulcan
error: The branch 'feature-vulcan' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-vulcan'.
```
　　销毁失败。Git友情提醒，feature-vulcan分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用命令git branch -D feature-vulcan。

　　现在我们强行删除：
```
$ git branch -D feature-vulcan
Deleted branch feature-vulcan (was 756d4af).
```
　　注意：开发一个新feature，最好新建一个分支；如果要丢弃一个没有被合并过的分支，可以通过git branch -D <name>强行删除。

### 10.推送分支
---
当你从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且远程仓库的默认名称是origin。

　　要查看远程库的信息，用git remote
 ```
$ git remote
origin
 ```
　　或者，用git remote -v显示更详细的信息：
 ```
$ git remote -v
 ```
推送分支
　　推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：
 ```
 git push origin master
 ```
　　如果要推送其他分支，比如dev，就改成
```
git push origin dev
```
* master分支是主分支，因此要时刻与远程同步；
* dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
* bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
* feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。
　　
### 11.多人协助
---
多人协作时，大家都会往master和dev分支上推送各自的修改。当你的同事也克隆一份此项目从远程库，默认情况下，只能看到本地的master分支。现在，你的同事要在dev分支上开发，就必须创建远程origin的dev分支到本地，于是他用这个命令创建本地dev分支：
```
$ git checkout -b dev origin/dev
```
　　现在，他就可以在dev上继续修改，然后，时不时地把dev分支push到远程，并且已经向origin/dev分支推送了他的提交，这时你也对同样的文件作了修改，并试图推送，推送失败，因为你的同事的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送，git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接。

　　这回git pull成功，但是是合并有冲突，需要手动解决，解决的方法和分支管理中的解决冲突完全一样。解决后，提交，再push：
```
$ git commit -m "merge & fix hello.py"
[dev adca45d] merge & fix hello.py
$ git push origin dev
```
　　因此，多人协作的工作模式通常是这样：

* 首先，可以试图用git push origin branch-name推送自己的修改；
* 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
* 如果合并有冲突，则解决冲突，并在本地提交；
* 没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！
　　如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。
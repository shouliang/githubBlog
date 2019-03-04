title: 入门—二.进程管理
date: 2014-01-12 14:45:23
tags: [Node.js]
---

### 1.简介
---
process是一个全局内置对象，可以在代码中的任何位置访问此对象，这个对象代表我们的node.js代码宿主的操作系统进程对象。

使用process对象可以截获进程的异常、退出等事件，也可以获取进程的当前目录、环境变量、内存占用等信息，还可以执行进程退出、工作目录切换等操作。

process对象的一些常用方法。

### 2.cwd
---
当我们想要查看应用程序当前目录时，可以使用cwd函数，使用语法如下：
```javascript
process.cwd();
```

### 3.chdir
---
如果需要改变应用程序目录，就要使用chdir函数了，它的用法如下：
```javascript
process.chdir("目录");
```

### 4.stdout
---
stdout是标准输出流，它是干什么的呢？请下看下面的示例：
```javascript
console.log = function(d){
    process.stdout.write(d+'\n');
}
```
没错，它的作用就是将内容打印到输出设备上，console.log就是封装了它。

### 5.stderr
---
stderr是标准错误流，和stdout的作用差不多，不同的是它是用来打印错误信息的，我们可以通过它来捕获错误信息，基本使用方法如下：
```javascript
process.stderr.write(输入内容);
```

### 6.stdin
---
stdin是进程的输入流,我们可以通过注册事件的方式来获取输入的内容，如下：
```javascript
process.stdin.on('readable', function() {
  var chunk = process.stdin.read();
  if (chunk !== null) {
    process.stdout.write('data: ' + chunk);
  }
});
```
示例中的chunk就是输入流中的内容。

### 7.exit
---
如果你需要在程序内杀死进程，退出程序，可以使用exit函数，示例如下：
```javascript
process.exit(code);
```
参数code为退出后返回的代码，如果省略则默认返回0；

### 8.注册事件
---
前面讲到如何在输入流中打印信息，当我们需要获取stdout内容的时候应该怎么做呢？请看如下的示例：
```javascript
process.stdout.on('data',function(data){
   console.log(data);
});
```
为stdout注册data事件，我们就可以拿到它输出的内容了。

### 9.设置编码
---
在我们的输入输出的内容中有中文的时候，可能会乱码的问题，这是因为编码不同造成的，所以在这种情况下需要为流设置编码，如下示例：
```javascript
process.stdin.setEncoding(编码);
process.stdout.setEncoding(编码);
process.stderr.setEncoding(编码);
```
常用的编码格式有"utf8"等


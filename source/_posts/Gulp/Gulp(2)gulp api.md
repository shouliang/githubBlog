title: 二.gulp api
date: 2015-04-07 15:45:23
tags: [Gulp]
---

### 1.gulp的工作方式
---
在介绍gulp API之前，我们首先来说一下gulp.js工作方式。在gulp中，使用的是Nodejs中的stream(流)，首先获取到需要的stream，然后可以通过stream的pipe()方法把流导入到你想要的地方，比如gulp的插件中，经过插件处理后的流又可以继续导入到其他插件中，当然也可以把流写入到文件中。所以gulp是以stream为媒介的，它不需要频繁的生成临时文件，这也是我们应用gulp的一个原因。

　　gulp的使用流程一般是：首先通过gulp.src()方法获取到想要处理的文件流，然后把文件流通过pipe方法导入到gulp的插件中，最后把经过插件处理后的流再通过pipe方法导入到gulp.dest()中，gulp.dest()方法则把流中的内容写入到文件中。例如：
```
var gulp = require('gulp');
gulp.src('script/jquery.js')         // 获取流的api
    .pipe(gulp.dest('dist/foo.js')); // 写放文件的api
```

### 2.globs的匹配规则
---
gulp用到的globs的匹配规则以及一些文件匹配技巧。

　　gulp内部使用了node-glob模块来实现其文件匹配功能。我们可以使用下面这些特殊的字符来匹配我们想要的文件：
![](http://7xq1il.com1.z0.glb.clouddn.com/mkpattern.png)
下面以例子来加深理解

\* 能匹配 a.js,x.y,abc,abc/,但不能匹配a/b.js

\*.\* 能匹配 a.js,style.css,a.b,x.y

\*/\*/\*.js 能匹配 a/b/c.js,x/y/z.js,不能匹配a/b.js,a/b/c/d.js

\*\* 能匹配 abc,a/b.js,a/b/c.js,x/y/z,x/y/z/a.b,能用来匹配所有的目录和文件

\*\*/\*.js 能匹配 foo.js,a/foo.js,a/b/foo.js,a/b/c/foo.js

a/\*\*/z 能匹配 a/z,a/b/z,a/b/c/z,a/d/g/h/j/k/z

a/\*\*b/z 能匹配 a/b/z,a/sb/z,但不能匹配a/x/sb/z,因为只有单**单独出现才能匹配多级目录

?.js 能匹配 a.js,b.js,c.js

a?? 能匹配 a.b,abc,但不能匹配ab/,因为它不会匹配路径分隔符

[xyz].js 只能匹配 x.js,y.js,z.js,不会匹配xy.js,xyz.js等,整个中括号只代表一个字符

[^xyz].js 能匹配 a.js,b.js,c.js等,不能匹配x.js,y.js,z.js
### 3.获取流
---
#### src

　gulp.src()方法正是用来获取流的，但要注意这个流里的内容不是原始的文件流，而是一个虚拟文件对象流(Vinyl files)，这个虚拟文件对象中存储着原始文件的路径、文件名、内容等信息。其语法为：
```
gulp.src(globs[, options]);
```
globs参数是文件匹配模式(类似正则表达式)，用来匹配文件路径(包括文件名)，当然这里也可以直接指定某个具体的文件路径。当有多个匹配模式时，该参数可以为一个数组;类型为String或 Array。

当有多种匹配模式时可以使用数组
```
//使用数组的方式来匹配多种文件
gulp.src(['js/\*.js','css/\*.css','*.html'])
```
options为可选参数。以下为options的选项参数:

__options.buffer__

类型： Boolean 默认值： true

　　如果该项被设置为 false，那么将会以 stream 方式返回 file.contents 而不是文件 buffer 的形式。这在处理一些大文件的时候将会很有用。注意：插件可能并不会实现对 stream 的支持。

__options.read__

类型： Boolean 默认值： true

如果该项被设置为 false， 那么 file.contents 会返回空值（null），也就是并不会去读取文件。

__options.base__

类型： String ， 设置输出路径以某个路径的某个组成部分为基础向后拼接。

如, 请想像一下在一个路径为 client/js/somedir 的目录中，有一个文件叫 somefile.js ：
```javascript
// 匹配 'client/js/somedir/somefile.js'现在 'base' 的值为 'client/js/'
gulp.src('client/js/\*\*/\*.js')  
  .pipe(minify())
  .pipe(gulp.dest('build')); 
gulp.src('client/js/\*\*/*.js', { base: 'client' }) 
  .pipe(minify())
  .pipe(gulp.dest('build'));  
```

### 4.写文件
---
#### dest 
gulp.dest()方法是用来写文件的，其语法为：
```
gulp.dest(path[,options])
```
path为写入文件的路径；

options为一个可选的参数对象，以下为选项参数：

options.cwd

类型： String 默认值： process.cwd()

输出目录的 cwd 参数，只在所给的输出目录是相对路径时候有效。

options.mode

类型： String 默认值： 0777

八进制权限字符，用以定义所有在输出目录中所创建的目录的权限。
```
var gulp = require('gulp');
gulp.src('script/jquery.js')　       // 获取流
    .pipe(gulp.dest('dist/foo.js')); // 写放文件
```
下面再说说生成的文件路径与我们给_gulp.dest()_方法传入的路径参数之间的关系。 　　_gulp.dest(path)_生成的文件路径是我们传入的_path_参数后面再加上_gulp.src()_中有通配符开始出现的那部分路径。例如：

```
var gulp = reruire('gulp'); //有通配符开始出现的那部分路径为 **/*.js
gulp.src('script/**/*.js')
    .pipe(gulp.dest('dist')); //最后生成的文件路径为 dist/**/*.js
//如果 **/*.js 匹配到的文件为 jquery/jquery.js ,则生成的文件路径为 dist/jquery/jquery.js
```

用gulp.dest()把文件流写入文件后，文件流仍然可以继续使用。

### 5.监视文件
---
#### watch
gulp.watch()用来监视文件的变化，当文件发生变化后，我们可以利用它来执行相应的任务，例如文件压缩等。其语法为

gulp.watch(glob[, opts], tasks); 

glob 为要监视的文件匹配模式，规则和用法与gulp.src()方法中的glob相同。 opts 为一个可选的配置对象，通常不需要用到。 tasks 为文件变化后要执行的任务，为一个数组。
```　
gulp.task('uglify',function(){
  //do something
});
gulp.task('reload',function(){
  //do something
});
gulp.watch('js/**/*.js', ['uglify','reload']);
　　gulp.watch()还有另外一种使用方式：　
gulp.watch(glob[, opts, cb]);
```
glob和opts参数与第一种用法相同;

cb参数为一个函数。每当监视的文件发生变化时，就会调用这个函数,并且会给它传入一个对象，该对象包含了文件变化的一些信息，type属性为变化的类型，可以是added,changed,deleted；path属性为发生变化的文件的路径。

gulp.watch('js/\*\*/\*.js', function(event){
    console.log(event.type); //变化类型 added为新增,deleted为删除，changed为改变 
    console.log(event.path); //变化的文件的路径
}); 

### 6.定义任务
---
#### task 
gulp.task方法用来定义任务，内部使用的是Orchestrator(用于排序、执行任务和最大并发依赖关系的模块)，其语法为：
```
gulp.task(name[, deps], fn)
```
name 为任务名；

deps 是当前定义的任务需要依赖的其他任务，为一个数组。当前定义的任务会在所有依赖的任务执行完毕后才开始执行。如果没有依赖，则可省略这个参数；

fn 为任务函数，我们把任务要执行的代码都写在里面。该参数也是可选的。

当你定义一个简单的任务时，需要传入任务名字和执行函数两个属性。
```
gulp.task('greet', function () {
   console.log('Hello world!');
});
```
执行gulp greet的结果就是在控制台上打印出“Hello world”。

你也可以定义一个在gulp开始运行时候默认执行的任务，并将这个任务命名为“default”：
```
gulp.task('default', function () {
   // Your default task
});
```
前面已经介绍了gulp.task的语法，但是当有多个任务时，需要知道怎么来控制任务的执行顺序。

可以通过任务依赖来实现。例如我想要执行one,two,three这三个任务，那我们就可以定义一个空的任务，然后把那三个任务当做这个空的任务的依赖就行了：

//只要执行default任务，就相当于把one,two,three这三个任务执行了
```
gulp.task('default',['one','two','three']);
```
　　如果任务相互之间没有依赖，任务就会按你书写的顺序来执行，如果有依赖的话则会先执行依赖的任务。但是如果某个任务所依赖的任务是异步的，就要注意了，gulp并不会等待那个所依赖的异步任务完成，而是会接着执行后续的任务。例如：
```　　
gulp.task('one',function(){
  //one是一个异步执行的任务
  setTimeout(function(){
    console.log('one is done')
  },5000);
});
//two任务虽然依赖于one任务,但并不会等到one任务中的异步操作完成后再执行
gulp.task('two',['one'],function(){
  console.log('two is done');
});
```
上面的例子中我们执行two任务时，会先执行one任务，但不会去等待one任务中的异步操作完成后再执行two任务，而是紧接着执行two任务。所以two任务会在one任务中的异步操作完成之前就执行了。

那如果我们想等待异步任务中的异步操作完成后再执行后续的任务，该怎么做呢？

有三种方法可以实现：
第一：在异步操作完成后执行一个回调函数来通知gulp这个异步任务已经完成,这个回调函数就是任务函数的第一个参数。

```
gulp.task('one',function(cb){ //cb为任务函数提供的回调，用来通知任务已经完成
  //one是一个异步执行的任务
  exec(function(){
    console.log('one is finish');
    cb();  //执行回调，表示这个异步任务已经完成
  },5000);
}); 
//这时two任务会在one任务中的异步操作完成后再执行
gulp.task('two',['one'],function(){
  console.log('two is finish');
});
```
第二：定义任务时返回一个流对象。适用于任务就是操作gulp.src获取到的流的情况。
```
gulp.task('one',function(cb){
  var stream = gulp.src('client/**/*.js')
      .pipe(exec()) //exec()中有某些异步操作
      .pipe(gulp.dest('build'));
    return stream;
});
gulp.task('two',['one'],function(){
  console.log('two is done');
});
```
第三：返回一个promise对象，例如
```
var Q = require('q');
gulp.task('one', function() {
  var deferred = Q.defer();
  setTimeout(function() {    // 执行异步的操作
    deferred.resolve();
  }, 1);
  return deferred.promise;
});
gulp.task('two',['one'],function(){
  console.log('two is done');
});　　
```
### 7.执行文件
---
#### run
　gulp.run()表示要执行的任务。可能会使用单个参数的形式传递多个任务。如下代码：
```
gulp.task('end',function(){
gulp.run('task1','task3','task2');
});
```
注意：任务是尽可能多的并行执行的，并且可能不会按照指定的顺序运行。

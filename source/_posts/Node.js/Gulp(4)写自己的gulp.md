title: 四.写自己的gulp
date: 2015-04-12 19:45:23
tags: [Node.js]
---

### 1.项目需求
---
我们将创建一个自己的gulp，具体的需求是通过gulp把我们自己所编写的JS文件合并压缩、CSS文件进行压缩后，并且生成新的文件。我们所需要的插件为：gulp-minify-css gulp-concat gulp-uglify gulp-rename del 如下图所示，完成后的项目目录结构：
![](http://7xq1il.com1.z0.glb.clouddn.com/mkgulp_project.gif)

### 2.创建项目
---
首先我们先来创建一个名为project的目录，然后进入到该目录下面，再将gulp安装到我们项目的目录中，然后在该目录下新建一个名称为gulpfile.js的文件。安装好后的目录结构为：
![](http://7xq1il.com1.z0.glb.clouddn.com/mkgulp_project_create.gif)
在该目录下再创建一个src目录，来存放源JS与CSS文件。建立完成后，再src目录分别建立两个js文件与一个CSS文件。完成后的目录结构为：
![](http://7xq1il.com1.z0.glb.clouddn.com/mkgulp_project_create1.gif)
### 3.安装插件
---
　　根据我们项目的需求，安装所需要的插件，可以通过"npm install 插件名" 来安装插件。安装完成后的目录结构如图所示。
![](http://7xq1il.com1.z0.glb.clouddn.com/mkgulp_project_plugin.gif)
然后打开gulpfile.js，将我们所用到的插件引用到我们项目中，代码如下:
```
var gulp = require('gulp'),
    minifycss = require('gulp-minify-css'),  //CSS压缩
    concat = require('gulp-concat'),         // 文件合并
    uglify = require('gulp-uglify'),         //js压缩插件
    rename = require('gulp-rename'),         // 重命名
    del = require('del');                    // 文件删除
```
### 4.编写代码
---
上一节中已经完成了对插件的引用，下面就开始我们的代码编写，可以通过gulp.start()方法来开始执行我们的任务。

1.gulp默认的执行任务是 "default"，当然你也可以指定别的名称，然后通过"gulp 任务名称" 来运行。
```
gulp.task('default',  function() {
    gulp.start('clean','minifycss', 'minifyjs');  // 要执行的任务
});
```
2.CSS压缩
```
gulp.task('minifycss', function() {
    return gulp.src('src/*.css')                  //压缩的文件
         .pipe(minifycss())                       //执行压缩
         .pipe(gulp.dest('minified/css'));        //输出文件夹
});
```
3.JS 合并压缩
```
gulp.task('minifyjs', function() {
    return gulp.src('src/*.js')
        .pipe(concat('main.js'))                  //合并所有js到main.js
        .pipe(gulp.dest('minified/js'))           //输出main.js到文件夹
        .pipe(rename({suffix: '.min'}))           //rename压缩后的文件名
        .pipe(uglify())                           //压缩
        .pipe(gulp.dest('minified/js'));          //输出
});
```
4.执行压缩前，先删除目录里的内容
```
gulp.task('clean', function(cb) {
    del(['minified/css', 'minified/js'], cb)
});
```
好了，这样我们的代码就完成了。

### 5.运行项目
---
前面我们已经编写完成了代码，在命令行中先转到project目录下，就可以输入gulp命令来运行本项目了，刷新project目录看看会出现什么结果呢。运行完成后的目录如下图：
![](http://7xq1il.com1.z0.glb.clouddn.com/mkgulp_project1.gif)

运行过程中的消息如下图所示：

![](http://7xq1il.com1.z0.glb.clouddn.com/mkgulp_task_process.gif)
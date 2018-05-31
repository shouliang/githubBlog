title: 三.gulp 常用插件
date: 2015-04-09 15:45:23
tags: [Gulp]
---

### 1.插件安装
---
在我们编写gulp代码时候，需要用到一些gulp的插件，可以通过npm install --save-dev 插件名称 来安装。如下代码来安装自动加载插件：

npm install --save-dev gulp-load-plugins
　　要使用gulp的插件，首先得用require来把插件加载进来。
```
var gulp=require('gulp'),
    plugins=require('gulp-load-plugins')(),
    uglify = require('gulp-uglify'),
    minifyHtml = require('gulp-minify-html'),
    rename = require('gulp-rename');
```
　　gulp的插件有很多种，后面介绍几个插件的用法。如还想了解更多插件，请查阅相关资料。

### 2.自动加载
---
ulp-load-plugins这个插件能自动帮你加载package.json文件里的gulp插件。例如假设你的package.json文件里的依赖是这样的:
```
{
  "devDependencies": {
    "gulp": "~3.6.0",
    "gulp-rename": "~1.2.0",
    "gulp-ruby-sass": "~0.4.3",
    "gulp-load-plugins": "~0.5.1"
  }
}
```
然后我们可以在gulpfile.js中使用gulp-load-plugins来帮我们加载插件：
```
var gulp = require('gulp');
//加载gulp-load-plugins插件，并马上运行它
var plugins = require('gulp-load-plugins')();
```
　　然后我们要使用gulp-rename和gulp-ruby-sass这两个插件的时候，就可以使用plugins.rename和plugins.rubySass来代替了,也就是原始插件名去掉gulp-前缀，之后再转换为驼峰命名。
### 3.重命名
---
gulp-rename插件用来重命名文件流中的文件。用gulp.dest()方法写入文件时，文件名使用的是文件流中的文件名，如果要想改变文件名，那可以在之前用gulp-rename插件来改变文件流中的文件名。
```
var gulp = require('gulp'),
    rename = require('gulp-rename'),
    uglify = require("gulp-uglify"); 
gulp.task('rename', function () {
    gulp.src('src/1.js')
    .pipe(uglify())           //压缩
    .pipe(rename('1.min.js')) //会将1.js重命名为1.min.js
    .pipe(gulp.dest('js'));
});
```

### 4.js文件压缩
---
gulp-uglify插件用来压缩js文件。
```
var gulp = require('gulp'),
    uglify = require("gulp-uglify"); 
gulp.task('minify-js', function () {
    gulp.src('src/*.js')          // 要压缩的js文件
    .pipe(uglify())              //使用uglify进行压缩
    .pipe(gulp.dest('dist/js')); //压缩后的路径
});
```
### 5.css文件压缩
---
gulp-minify-css插件用来压缩css文件。
```
var gulp = require('gulp'),
    minifyCss = require("gulp-minify-css");
gulp.task('minify-css', function () {
    gulp.src('src/*.css') // 要压缩的css文件
    .pipe(minifyCss())    //压缩css
    .pipe(gulp.dest('dist/css'));
});
```
### 6.html文件压缩
---
gulp-minify-html插件用来压缩html文件。
```
var gulp = require('gulp'),
    minifyHtml = require("gulp-minify-html"); 
gulp.task('minify-html', function () {
    gulp.src('src/*.html') // 要压缩的html文件
    .pipe(minifyHtml())    //压缩
    .pipe(gulp.dest('dist/html'));
});
```
### 7.js代码检查
---
使用gulp-jshint插件，用来检查js代码。
```
var gulp = require('gulp'),
    jshint = require("gulp-jshint");
gulp.task('jsLint', function () {
    gulp.src('src/*.js')
    .pipe(jshint())
    .pipe(jshint.reporter()); // 输出检查结果
});
```
### 8.文件合并
---
使用gulp-concat插件，用来把多个文件合并为一个文件,我们可以用它来合并js或css文件等。
```
var gulp = require('gulp'),
    concat = require("gulp-concat");
gulp.task('concat', function () {
    gulp.src('src/*.js')     //要合并的文件
    .pipe(concat('all.js'))  // 合并匹配到的js文件并命名为 "all.js"
    .pipe(gulp.dest('dist/js'));
});
```
### 9.图片压缩
---
可以使用gulp-imagemin插件来压缩jpg、png、gif等图片。
```
var gulp = require('gulp');
var imagemin = require('gulp-imagemin');
var pngquant = require('imagemin-pngquant'); //png图片压缩插件
gulp.task('default', function () {
    return gulp.src('src/images/*')
        .pipe(imagemin({
            progressive: true,
            use: [pngquant()] //使用pngquant来压缩png图片
        }))
        .pipe(gulp.dest('dist'));
});
```

### 10.自动刷新
---
使用gulp-livereload插件，当代码变化时，它可以帮我们自动刷新页面。
```
var gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload');
gulp.task('less', function() {
  gulp.src('less/\*.less')
    .pipe(less())
    .pipe(gulp.dest('css'))
    .pipe(livereload());
});
gulp.task('watch', function() {
  livereload.listen(); //要在这里调用listen()方法
  gulp.watch('less/*.less', ['less']);
});
```
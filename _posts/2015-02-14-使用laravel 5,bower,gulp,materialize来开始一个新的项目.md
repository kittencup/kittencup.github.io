---
layout: post
title: "使用laravel 5,bower,gulp,materialize来开始一个新的项目"
date: 2015-02-14
categories: php
---

使用laravel 5,bower,gulp,materialize来开始一个新的项目

[laravel 5](http://laravel.com/) 是一个为网页艺术家创造的，也是最流行的PHP框架之一

[bower](http://bower.io/) 是一个前端包管理器，类似于php的composer

[gulp](http://gulpjs.com/) 是一个前端自动化工具，提高你的前端开发效率

[materialize](http://materializecss.com/) 是一个基于Material Design(Google的视觉设计规范)的响应式设计前端框架

2.安装laravel 5 
--------
composer 安装

```
composer create-project laravel/laravel kittencup-laravel --prefer-dist
```


3.安装bower和gulp
--------

```
npm install -g bower  
npm install -g gulp  
```

4.设置bower
--------
在项目根目录创建bower.json，内容为

```
{
  "name": "kittencup-laravel ",
  "dependencies": {
    "jquery": ">=2.1.1",
    "materialize": ">=v0.95.2"
  }
}
```

项目的目录结构

```
kittencup-laravel  
- app
- bootstrap
- config
- database
- public
- resources
  - assets
  - lang
  - views
- storage
- tests
- vendor
- bower.json
- gulpfile.js
- package.json
```

对于开发阶段来讲 resources/assets 存放的是开发时需要用到的资源(less,sass,未压缩的js等)，bower安装依赖的路径是bower.json所在目录，所以需要设置下bower安装目录，在根目录创建.bowerrc，添加内容

```
{
  "directory": "resources/assets/vendor"
}
```

执行安装依赖

```
bower install
```
materialize,jquery被安装到文件夹resources/assets/vendor下

5.创建前端文件
--------
resources/assets/sass/site.sass
resources/assets/js/site.js

6.设置Gulp
--------
项目根目录如不存在package.json则创建，设置依赖

```
{
  "devDependencies": {
    "gulp": "^3.8.8",
    "gulp-less": "^1.3.6",
    "gulp-sass": "^1.0.0",
    "gulp-minify-css": "^0.3.10",
    "gulp-concat": "^2.4.1",
    "gulp-uglify": "^1.0.1",
    "gulp-rename": "^1.2.0",
    "gulp-phpunit": "^0.6.3",
    "gulp-notify-growl": "^1.0.2",
    "gulp-notify": "^1.8.0"
  }
}
```

执行安装依赖

```
npm install
```
依赖都会下载到node_modules

根目录未gulpfile.js则创建，设置任务

导入模块

```
var gulp = require('gulp'),
    less = require('gulp-less'),
    sass = require('gulp-sass'),
    minify = require('gulp-minify-css'),
    concat = require('gulp-concat'),
    uglify = require('gulp-uglify'),
    rename = require('gulp-rename'),
    notify = require('gulp-notify'),
    growl = require('gulp-notify-growl'),

```

任务路径变量，方便后面使用

```
var paths = {
    'dev': {
        'sass': './resources/assets/sass/',
        'js': './resources/assets/js/',
        'vendor': './resources/assets/vendor/'
    },
    'production': {
        'css': './public/assets/css/',
        'js': './public/assets/js/'
    }
};

```

创建css任务，编译sass并将多个css合并到/public/assets/css/site.css,然后进行压缩,改名为site.min.css

```
gulp.task('css', function() {
    return gulp.src([
        paths.dev.sass+'site.scss',
        paths.dev.vendor+'materialize/sass/materialize.scss'
    ]).pipe(sass())
      .pipe(concat('site.css'))
      .pipe(gulp.dest(paths.production.css))
      .pipe(minify({keepSpecialComments:0}))
      .pipe(rename({suffix: '.min'}))
      .pipe(gulp.dest(paths.production.css));
});
```

创建js任务，合并多个js到/public/assets/css/site.js,然后进行压缩,改名为site.min.js

```
gulp.task('js', function(){
    return gulp.src([
        paths.dev.vendor+'jquery/dist/jquery.js',
        paths.dev.vendor+'materialize/dist/js/materialize.js',
        paths.dev.js+'site.js'
    ]).pipe(concat('site.js'))
      .pipe(gulp.dest(paths.production.js));
      .pipe(uglify())
      .pipe(rename({suffix: '.min'}))
      .pipe(gulp.dest(paths.production.js));
});
```

创建watch任务，监控sass,js文件，有变化则自动执行css,js任务

```
gulp.task('watch', function() {
    gulp.watch(paths.dev.sass + '/*.scss', ['css']);
    gulp.watch(paths.dev.js + '/*.js', ['js']);
});
```

创建default任务，当执行default任务时，会执行上面定义的css,js,watch任务

```
gulp.task('default', ['css', 'js','watch']);
```

7.执行任务
--------

```
gulp default 
```

因为设置了watch任务，当改变sass或者js文件，则会自动触发上述的css和js任务

8.设置.gitignore
--------
如果你使用git来管理你的项目，请将 /node_modules 和 /resources/assets/vendor 添加到.gitignore文件，这样提交时就不会提交该文件夹










---
layout: post
title: "使用Yeoman构建Web App工作流 (一)"
date: 2015-02-27
categories: javascript
---

[Yeoman](http://yeoman.io/)是Google的团队和外部贡献者团队合作开发的，他的目标是通过[Yo](http://yeoman.io/),[Grunt](http://gruntjs.com/)和[Bower](http://bower.io/)为开发者创建一个易用Web应用的工作流。

2.Web应用开发流程
-------
1. 搭建目录结构文件夹 images,styles,scripts,vendor
2. 编写HTML，可能你会用[jade](http://jade-lang.com/)等模板引擎，那么需要编译成html
3. 编写css,可能你需要[Sass](http://sass-lang.com/)或[Less](http://lesscss.org/)等预处理来编写,那么需要编译成css，放入styles文件夹
4. 编写javascript,可能你也需要用[AtScript](https://docs.google.com/document/d/11YUzC-1d0V1-Q3V0fQ7KSit97HnZoKVygDxpWzEYW0U/preview?sle=true)，[TypeScript](http://www.typescriptlang.org/)，[CoffeeScript](http://coffeescript.org/)来编写,那么需要编译成js，放入scripts文件夹
5. 有时候你还需要用到第3方的库，比如jQuery,你需要去[jQuery官网](http://jquery.com)下载jquery.js，并放入vendor文件夹
6. 对js文件进行编码检查和单元测试
7. 开发完毕后，你需要针对css,js进行压缩合并，已减少网站打开时的请求次数和文件大小
8. 还需要对图片和SVG进行优化，减少文件大小

3.Yeoman可以做什么
-------
Yo
> 帮你构建目录结构或者文件,解决开发流程中(1)

bower
> 第3方组件管理,解决开发流程中(5)

grunt
> 自动化构建工具,解决开发流程中(2,3,4,6,7,8)

> 自动化构建工具也可以使用[Gulp](http://gulpjs.com/)

4.安装Yeoman
-------
安装yeoman,grunt,bower,-g 安装在全局范围内，任何地方都能调用到相应命令

```bash
$ npm install -g yo bower grunt-cli gulp
```

安装 generator-webapp ，一个Web应用的脚手架构建程序，由[HTML5 Boilerplate](http://html5boilerplate.com/), [jQuery](http://jquery.com/), [Modernizr](http://modernizr.com/), 和 [Bootstrap](http://twbs.github.io/bootstrap)构成

```bash
$ npm install -g generator-webapp
```

> 如果你想开发的是AngularJS的Web应用，你可以安装generator-angular构建程序，[查看更多的构建程序](http://yeoman.io/generators/)

5.使用Yeoman创建目录结构
-------
我的项目是Blog

```bash
$ mkdir /Users/Kittencup/WebServer/Blog
$ cd /Users/Kittencup/WebServer/Blog
$ yo webapp
```
根据需求选择上Bootstrap,Sass,Modernizr

安装完成后,Blog下生成的目录结构

![](http://www.kittencup.com/wp-content/uploads/2015/02/yo.jpg)

6.目录结构
-------
app 
> 源文件的存放目录,未压缩的js,css,scss等

bower_components 
> 使用bower安装的第3方组件的源代码,jquery,modernizr,bootstrap

node_modules
> grunt 需要用到的工具模块安装目录

test
> js 单元测试存放目录

.bowerrc
> bower的配置目录，配置了使用bower安装的第3方组件存放目录

.editorconfig
> 帮助开发者在不同的编辑器和IDE之间定义和维护一致的代码风格的配置，[具体查看](http://editorconfig.org)

.gitattributes
> git属性设置，具体查看git文档

.gitignore
> git提交时忽略的目录，如bower_components和node_modules等目录并不需要提交给git

.jshintrc
> jshint的配置文件，[具体查看](http://jshint.com/)

.yo-rc.json
> yo的配置文件

.bower.json
> bower的配置文件，根据配置安装第3方组件到bower_components文件夹下

.Gruntfile.json
> grunt的任务配置，根据任务配置自动化处理css,js等

.package.json
> grunt的配置文件，根据配置安装grunt的依赖组件到node_modules文件夹下

7.开发项目时
-------
在开发时，我们在app文件下进行开发，Grunt已经配置了开发任务

```bash
$ grunt serve
```
执行完命令后，创建tmp文件夹，存放经过编译后的scss文件，通过[grunt-contrib-connect](https://github.com/gruntjs/grunt-contrib-connect)模拟服务器打开app/index.html，并且通过[grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch)监听指定文件(html,css,js等)，如果文件内容改变，则通过[grunt-contrib-connect](https://github.com/gruntjs/grunt-contrib-connect)内的[livereload](https://github.com/gruntjs/grunt-contrib-watch#optionslivereload)功能通知浏览器，实现浏览器自动刷新

8.LiveReload
------
livereload功能需要配置浏览器上的扩展，[Chrome-liveReload](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei)

![](http://www.kittencup.com/wp-content/uploads/2015/02/liveload.jpg)


安装完毕后，在浏览器右上方开启liveReload

开启图案
![](http://www.kittencup.com/wp-content/uploads/2015/02/open-livereload.jpg)

这样LiveReload功能才能正常使用

9.开发完毕时
--------
开发完毕时，需要对开发的内容进行检测，测试，压缩合并等

```bash
$ grunt default
```
dist目录生成，经过压缩合并等的内容都在该目录下

10.下一章内容
--------
下一章我将会讲解grunt的每个任务，GruntFile过于庞大，需要分解GruntFile文件，使每个任务独立，方便配置。

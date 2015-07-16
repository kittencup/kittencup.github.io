---
layout: post
title: "使用Yeoman构建Web App工作流 (三)"
date: 2015-02-28
categories: javascript
---

本章主要讲分析grunt的任务流程,主要任务分2个，开发时任务server,和开发完毕后的打包任务default

2.Server任务
-----------

```javascript
grunt.registerTask('server', 'start the server and preview your app, --allow-remote for remote access', function (target) {
            if (grunt.option('allow-remote')) {
                grunt.config.set('connect.options.hostname', '0.0.0.0');
            }

            grunt.task.run([
                'clean:server',
                'wiredep',
                'concurrent:server',
                'autoprefixer',
                'connect:livereload',
                'watch'
            ]);
        });
```

允许通过命令参数--allow-remote来控制服务器hostname，默认是localhost

```bash
grunt server --allow-remote
```

1. clean:server
> 调用clean任务中的server子任务，删除.tmp文件夹

2. wiredep
> 通过HTML注视，来注入Bower所安装的第3方组件的js,css

```html
#app/index.html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
      <!-- bower:css -->
      <!-- endbower -->
  </head>
  <body>
      <!-- bower:js -->
      <!-- endbower -->
  </body>
</html>
```
> wiredep会根据bower.json的dependencies自动将相关的第3方组件css,js注入到html注释内，在wiredep.js里并未设置css配置，只设置了js，并且排除了bootstrap.js，所以会把bower依赖的js注入到<!-- bower:js --><!-- endbower --> 注释内

```html
#app/index.html
......
<!-- bower:js -->
<script src="bower_components/jquery/dist/jquery.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/affix.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/alert.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/button.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/carousel.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/collapse.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/dropdown.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/tab.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/transition.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/scrollspy.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/modal.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/tooltip.js"></script>
<script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/popover.js"></script>
<!-- endbower -->
......
```

3. concurrent:server
> 执行concurrent.js里server指定的独立内容，'sass:server'和'copy:styles'

4. sass:server
> 将app/sass内所有的sass编译成css,放入到tmp/styles下，仅供开发时使用

5. copy:styles
> 将app/styles内的css文件也复制到.tmp/styles下

6. autoprefixer
> 将tmp/styles下的所有ccs文件内的css3属性补上浏览器前缀，开发时不用在意不同浏览器属性的私有前缀问题

```scss
#app/sass/main.scss
body{
	transform:rotate(-3deg);
}
```
```css
#.tmp/css/main.css
body {
  -webkit-transform: rotate(-3deg);
      -ms-transform: rotate(-3deg);
          transform: rotate(-3deg); }
```

7. connect:livereload
> 为liveReload创建虚拟服务器，设置静态文件的根目录

8. watch
> 为一些文件创建监听，文件如果有变化(css和js修改了等)，重新执行某些生成任务，并通知浏览器刷新

3.Default任务
-----------
`````bash
default:
  - newer:jshint
  - test
  - build
`````

1. newer:jshint
> newer是一个grunt组建[grunt-newer](https://github.com/tschaub/grunt-newer),提高grunt效率，只执行那些修改过的代码的操作，newer:jshint是对那些修改过的js文件进行js检查

2. test
> 创建虚拟服务器，使用mocha进行线上测试

3. build

`````bash
build:
  - clean:build
  - wiredep
  - useminPrepare
  - concurrent:build
  - autoprefixer
  - concat
  - cssmin
  - uglify
  - copy:build
  - modernizr
  - rev
  - usemin
  - htmlmin
`````

4. clean:build
> 清空tmp和dist文件夹

5. wiredep
> 同Server任务中一样

6. useminPrepare
> 为usemin做准备

7. concurrent:build
> 解析sass,copy css,优化image,svg

8. autoprefixer
> 同Server任务中一样

9. concat
> 合并内容，此处根据usemin解析的数据来合并，自身没有需要合并的配置，所以没具体合并任务

10. cssmin
> css 压缩，同上

11. uglify
> js 压缩，同上

12. copy:build
> 将ico,font,htaccess复制到dist目录下

13. modernizr
> 将modernizr移动到dist目录，modernizr需要单独一个文件，所以不要和其他的合并在一起，build:js也有modernizr，不过因为没定义路径，所以找不到modernizr.js,但是因为build:js解析完以后，注释中间的js将移除，最后在通过grunt-modernizr来解析一个适合的modernizr

14. rev
> 将dist目录的的文件名hash

15. usemin
> 通过html的注释，做一系列合并压缩解析，并且修改html内的外链文件名为rev模块hash后的文件名

```html
#app/index.html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <!-- build:css(.) styles/vendor.css -->
      <!-- bower:css -->
      <!-- endbower -->
    <!-- endbuild -->

    <!-- build:css(.tmp) styles/main.css -->
    <link rel="stylesheet" href="styles/main.css">
    <!-- endbuild -->

    <!-- build:js scripts/vendor/modernizr.js -->
      <script src="bower_components/modernizr/modernizr.js"></script>
    <!-- endbuild -->
  </head>
  <body>
    <!-- build:js(.) scripts/vendor.js -->
      <!-- bower:js -->
      <script src="bower_components/jquery/dist/jquery.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/affix.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/alert.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/button.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/carousel.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/collapse.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/dropdown.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/tab.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/transition.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/scrollspy.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/modal.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/tooltip.js"></script>
      <script src="bower_components/bootstrap-sass-official/assets/javascripts/bootstrap/popover.js"></script>
      <!-- endbower -->
    <!-- endbuild -->
  </body>
</html>
```
> build:css(.tmp) styles/main.css 将build:css注释块内的css(这些href的根目录是.tmp文件夹)，合并压缩到dist/styles/main.css里
> build:js(.) scripts/vendor.js 将build:js注释块内的js(这些src的根目录是项目根目录文件夹)，合并压缩到scripts/vendor.js里

16. htmlmin

> 优化html,这里将html内的大写标签全部转换为小写







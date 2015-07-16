---
layout: post
title: "使用Yeoman构建Web App工作流 (二)"
date: 2015-02-28
categories: javascript
---

上一章节创建的 Gruntfile.js 配置已经400多行了，简单的自动化流程，也变成复杂的流程了，一种解决办法是使用[Gulp](http://gulpjs.com/)来代替Grunt，[Gulp](http://gulpjs.com/)相对Grunt更简单，另一种方式则是通过分解Gruntfile.js文件来解决

2.分解Gruntfile.js
-------
分解Gruntfile.js文件，将所有任务分解到单个文件中，需要用到[load-grunt-config模块](https://github.com/firstandthird/load-grunt-config)

```
$ npm install load-grunt-config --save-dev
```
--save-dev将load-grunt-config保存到package.json的devDependencies里

修改Gruntfile.js为

```javascript
 'use strict';

(function() {
    var path = require('path'),
        config = {
            app: 'app',
            dist: 'dist',
            tmp: '.tmp'
        };

    module.exports = function(grunt) {
        require('time-grunt')(grunt);

        require('load-grunt-config')(grunt, {
            configPath: path.join(process.cwd(), 'tasks'),
            data: {
                config: config,
                packageJson: require('./package.json')
            },
            init: true
        });
        
        // ..定义其他任务 YAML里无法定义的任务
         grunt.registerTask('test', function () {
            if (target !== 'watch') {
                grunt.task.run([
                    'clean:server',
                    'concurrent:test',
                    'autoprefixer'
                ]);
            }

            grunt.task.run([
                'connect:test',
                'mocha'
            ]);
        });

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
    };
})();

```
设置了configPath为任务文件夹为tasks(未设置的话默认是grunt文件夹)，所有的任务文件都放在tasks下，并传递给这些任务data数据，可能在任务中会使用到，并自动init

如果在tasks目录下设置aliases.(js|.json|yaml|coffee) 文件，load-grunt-config 使用里面配置来定义你的任务 (类似 ```grunt.registerTask('default', ['jshint']);```).

我在这里使用[YMAL](http://yaml.org/)格式来定义部分任务，有些任务定义时无法用YAML来实现，所以还是定义在Gruntfile.js内

tasks/aliases.yaml

```bash
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

default:
  - newer:jshint
  - test
  - build
```

3.tasks目录结构
-------

![](http://www.kittencup.com/wp-content/uploads/2015/02/tasks.jpg)

4.用到的Grunt模块
-------

[time-grunt](https://github.com/sindresorhus/time-grunt)
> 显示Grunt任务的执行时间

[autoprefixer](https://github.com/nDmitry/grunt-autoprefixer)
> 为CSS3自动添加前缀

```javascript
module.exports = {
    options: {
        browsers: ['> 1%', 'last 2 versions', 'Firefox ESR', 'Opera 12.1']
    },
    dist: {
        files: [{
            expand: true,
            cwd: '.tmp/styles/',
            src: '{,*/}*.css',
            dest: '.tmp/styles/'
        }]
    }
}
```

[clean](https://github.com/gruntjs/grunt-contrib-clean)
> 清空文件夹

```javascript
module.exports = {
    build: {
        files: [{
            dot: true,
            src: [
                '.tmp',
                '<%= config.dist %>/*',
                '!<%= config.dist %>/.git*'
            ]
        }]
    },
    server: '.tmp'
};
```

[concat](https://github.com/gruntjs/grunt-contrib-concat)
> 合并文件

```javascript
module.exports = {
    build: {}
}
```

[concurrent](https://github.com/sindresorhus/grunt-concurrent)
> 定义任务，任务可執行其他grunt任务

```javascript
module.exports = {
    server: [
        'sass:server',
        'copy:styles'
    ],
    test: [
        'copy:styles'
    ],
    build: [
        'sass',
        'copy:styles',
        'imagemin',
        'svgmin'
    ]
};
```

[connect](https://github.com/gruntjs/grunt-contrib-connect)
> 启动Web服务器，还提供liveReload功能

```javascript
module.exports = function(grunt,data){
    return {
        options: {
            port: 9000,
            open: true,
            livereload: 35729,
            // Change this to '0.0.0.0' to access the server from outside
            hostname: 'localhost'
        },
        livereload: {
            options: {
                middleware: function(connect) {

                    return [
                        connect.static('.tmp'),
                        connect().use('/bower_components', connect.static('./bower_components')),
                        connect.static(data.config.app)
                    ];
                }
            }
        },
        test: {
            options: {
                open: false,
                port: 9001,
                middleware: function (connect,grunt) {
                    return [
                        connect.static('.tmp'),
                        connect.static('test'),
                        connect().use('/bower_components', connect.static('./bower_components')),
                        connect.static(data.config.app)
                    ];
                }
            }
        }
    };
}
```

[copy](https://github.com/gruntjs/grunt-contrib-copy)
> 复制文件和文件夹

```javascript
module.exports = {
    build: {
        files: [{
            expand: true,
            dot: true,
            cwd: '<%= config.app %>',
            dest: '<%= config.dist %>',
            src: [
                '*.{ico,png,txt}',
                'images/{,*/}*.webp',
                '{,*/}*.html',
                'styles/fonts/{,*/}*.*'
            ]
        }, {
            src: 'node_modules/apache-server-configs/dist/.htaccess',
            dest: '<%= config.dist %>/.htaccess'
        }, {
            expand: true,
            dot: true,
            cwd: '.',
            src: 'bower_components/bootstrap-sass-official/assets/fonts/bootstrap/*',
            dest: '<%= config.dist %>'
        }]
    },
    styles: {
        expand: true,
        dot: true,
        cwd: '<%= config.app %>/styles',
        dest: '.tmp/styles/',
        src: '{,*/}*.css'
    }
}
```

[cssmin](https://github.com/gruntjs/grunt-contrib-cssmin)
> 压缩css

```javascript
module.export = {
    build:{}
}
```

[htmlmin](https://github.com/gruntjs/grunt-contrib-htmlmin)
> 压缩和美化html

```javascript
module.exports = {
    build: {
        options: {
            caseSensitive:true
        },
        files: [{
            expand: true,
            cwd: '<%= config.dist %>',
            src: '{,*/}*.html',
            dest: '<%= config.dist %>'
        }]
    }
}
```

[imagemin](https://github.com/gruntjs/grunt-contrib-imagemin)
> 压缩图片

```javascript
module.exports = {
    build: {
        files: [{
            expand: true,
            cwd: '<%= config.app %>/images',
            src: '{,*/}*.{gif,jpeg,jpg,png}',
            dest: '<%= config.dist %>/images'
        }]
    }
}
```
[jshint](https://github.com/gruntjs/grunt-contrib-jshint)
> 验证js文件

```javascript
module.exports = {
    options: {
        jshintrc: '.jshintrc',
        reporter: require('jshint-stylish')
    },
    all: [
        'Gruntfile.js',
        '<%= config.app %>/scripts/{,*/}*.js',
        '!<%= config.app %>/scripts/vendor/*',
        'test/spec/{,*/}*.js'
    ]
}
```

[mocha](https://github.com/kmiyashiro/grunt-mocha)
> 服务器端的[mocha](http://mochajs.org/)单元测试

```javascript
module.exports = {
    all: {
        options: {
            run: true,
            urls: ['http://<%= connect.test.options.hostname %>:<%= connect.test.options.port %>/index.html']
        }
    }
}
```

[modernizr](https://github.com/Modernizr/grunt-modernizr)
> 检测浏览器的功能

```javascript
module.exports = {
    build: {
        devFile: 'bower_components/modernizr/modernizr.js',
        outputFile: '<%= config.dist %>/scripts/vendor/modernizr.js',
        files: {
            src: [
                '<%= config.dist %>/scripts/{,*/}*.js',
                '<%= config.dist %>/styles/{,*/}*.css',
                '!<%= config.dist %>/scripts/vendor/*'
            ]
        },
        uglify: true
    }
}
```

[rev](https://github.com/cbas/grunt-rev)
> 修改文件名为文件内容的哈希值

```javascript
module.exports = {
    build: {
        files: {
            src: [
                '<%= config.dist %>/scripts/{,*/}*.js',
                '<%= config.dist %>/styles/{,*/}*.css',
                '<%= config.dist %>/images/{,*/}*.*',
                '<%= config.dist %>/styles/fonts/{,*/}*.*',
                '<%= config.dist %>/*.{ico,png}'
            ]
        }
    }
}
```

[sass](https://github.com/sindresorhus/grunt-sass)
> 编译SASS

```javascript
module.exports = {
    options: {
        sourceMap: true,
        includePaths: ['bower_components']
    },
    server: {
        files: [{
            expand: true,
            cwd: '<%= config.app %>/styles',
            src: ['*.{scss,sass}'],
            dest: '.tmp/styles',
            ext: '.css'
        }]
    }
}
```
[svgmin](https://github.com/sindresorhus/grunt-svgmin)
> 压缩SVG

```javascript
module.exports = {
    build: {
        files: [{
            expand: true,
            cwd: '<%= config.app %>/images',
            src: '{,*/}*.svg',
            dest: '<%= config.dist %>/images'
        }]
    }
}
```javascript
[uglify](https://github.com/gruntjs/grunt-contrib-uglify)
> 压缩js

```javascript
module.exports = {
    build:{}
}
```

[usemin和useminPrepare](https://github.com/yeoman/grunt-usemin)
> 使用HTML内的注视语法，来压缩或合并css,js

usemin.js

```javascript
module.exports = {
    options: {
        assetsDirs: [
            '<%= config.dist %>',
            '<%= config.dist %>/images',
            '<%= config.dist %>/styles'
        ]
    },
    html: ['<%= config.dist %>/{,*/}*.html'],
    css: ['<%= config.dist %>/styles/{,*/}*.css']
}
```

useminPrepare.js

```javascript
module.exports = {
    options: {
        dest: '<%= config.dist %>'
    },
    html: '<%= config.app %>/index.html'
}
```

[watch](https://github.com/gruntjs/grunt-contrib-watch)
> 监听文件变化执行对应任务

```javascript
module.exports = {
    bower: {
        files: ['bower.json'],
        tasks: ['wiredep']
    },
    js: {
        files: ['<%= config.app %>/scripts/{,*/}*.js'],
        tasks: ['jshint'],
        options: {
            livereload: true
        }
    },
    jstest: {
        files: ['test/spec/{,*/}*.js'],
        tasks: ['test:watch']
    },
    gruntfile: {
        files: ['Gruntfile.js']
    },
    sass: {
        files: ['<%= config.app %>/styles/{,*/}*.{scss,sass}'],
        tasks: ['sass:server', 'autoprefixer']
    },
    styles: {
        files: ['<%= config.app %>/styles/{,*/}*.css'],
        tasks: ['newer:copy:styles', 'autoprefixer']
    },
    livereload: {
        options: {
            livereload: '<%= connect.options.livereload %>'
        },
        files: [
            '<%= config.app %>/{,*/}*.html',
            '.tmp/styles/{,*/}*.css',
            '<%= config.app %>/images/{,*/}*'
        ]
    }
}
```

[wiredep](https://github.com/stephenplusplus/grunt-wiredep)
> 通过HTML注视，来注入[Bower](http://bower.io)所安装的第3方组件的js,css

```javascript
module.exports = {
    app: {
        ignorePath: /^\/|\.\.\//,
        src: ['<%= config.app %>/*.html'],
        exclude: ['bower_components/bootstrap-sass-official/assets/javascripts/bootstrap.js']
    },
    sass: {
        src: ['<%= config.app %>/styles/{,*/}*.{scss,sass}'],
        ignorePath: /(\.\.\/){1,2}bower_components\//
    }
};
```

这样就将Gruntfile.js文件分解成一个个任务，有些任务配置在发开过程中基本不需要重新配置，通过文件名很容易定位到对应任务

5.使用方法
--------
开发时

```bash
$ grunt serve
```

开发完毕时

```bash
$ grunt default
```

6.下一章内容
--------
下一章我将分析任务中每一步的流程。



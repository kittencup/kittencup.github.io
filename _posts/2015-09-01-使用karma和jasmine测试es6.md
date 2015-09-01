---
layout: post
title: "使用karma和jasmine测试es6"
date: 2015-09-01
categories: javascript
---

项目地址
-----------
[https://github.com/kittencup/karma-jasmine-es6](https://github.com/kittencup/karma-jasmine-es6)

介绍
-----------
[Karma](http://karma-runner.github.io/) 用来自动化完成单元测试 [Jasmine](http://jasmine.github.io/) 用来做单元测试

package.json
-----------

```javascript
{
    "devDependencies": {
        "karma": "^0.13.9",
        "karma-babel-preprocessor": "^5.2.2",
        "karma-chrome-launcher": "^0.2.0",
        "karma-jasmine": "^0.3.6",
        "karma-requirejs": "^0.2.2"
      }
}
```

#### karma-babel-preprocessor

浏览器不能直接运行es6,在测试之前将es6转换成es5

#### karma-chrome-launcher

chrome 浏览器启动器，如果用其他浏览器测试，可安装对应的启动器

#### karma-jasmine,karma-requirejs

karma中需要用到的模块，通过babel将es6的module转换成amd模块，所以在这里使用requirejs

karma.conf
---------

```javascript
module.exports = function (config) {
    config.set({

            // files的路径
            basePath: './',

            // karma中使用的框架
            // 其他可用框架: https://npmjs.org/browse/keyword/karma-adapter
            frameworks: ['jasmine', 'requirejs'],

            // 需要加载到浏览器的js
            // 在这里使用included:false 禁止加载进浏览器
            // es6模块会通过babel转换为amd模块，通过requirejs按需加载，所以不需要在浏览器中引入这些文件
            // test-main.js是用来配置requirejs的，所以需要加载到浏览器中
            files: [
                {pattern: 'src/**/*.js', included: false},
                {pattern: 'test/**/*Spec.js', included: false},
                'test/test-main.js'
            ],

            
            // 在处理之前通过babel将es6模块转换成amd模块
            // 其他可用preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
            preprocessors: {
                'src/**/*.js': ['babel'],
                'test/**/!(test-main).js': ['babel']
            },
            babelPreprocessor: {
                options: {
                    sourceMap: 'inline',
                    modules: 'amd'
                }
            },

            // 自动观察文件变化
            autoWatch: true,

            // 启动哪些浏览器来运行测试
            // 其他浏览器启动程序: https://npmjs.org/browse/keyword/karma-launcher
            browsers: ['Chrome'],

            // 关闭单次运行
            singleRun: false
        }
    )
};
```

test-main.js
-----------

```javascript
var tests = [];
for (var file in window.__karma__.files) {
    if (/Spec\.js$/.test(file)) {
        tests.push(file);
    }
}

requirejs.config({
    // karma serve 默认是/base ，我们这里需要测试的文件都在/base/src下
    baseUrl: '/base/src',

    paths: {

    },

    shim: {

    },

    // 为浏览器加载所有的测试文件
    deps: tests,

    // requirejs加载完后启动karma
    callback: window.__karma__.start
});
```


---
layout: post
title: "Mac 下安装 Xdebug"
date: 2013-10-31
categories: ghost
---

###通过brew 安装 xdebug

```bash
brew install xdebug
```

###配置php.ini

```bash
zend_extension=/usr/local/Cellar/php54-xdebug/2.2.3/xdebug.so
;开启对函数调用的监测信息用文件格式输出
xdebug.auto_trace=1
;监测函数调用的参数
xdebug.collect_params=1
;监测函数的返回值
xdebug.collect_return=1
;检测调用的输出信息保存在
xdebug.trace_output_dir=""
;开启效能检测
xdebug.profiler_enable=1;
;性能信息保存在
xdebug.profiler_output_dir=""
```


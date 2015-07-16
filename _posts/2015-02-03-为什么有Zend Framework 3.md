---
layout: post
title: "为什么有Zend Framework 3?"
date: 2015-02-03
categories: php
---

### Zend Framework 2 的问题

Zend Framework 2的迄今为止最详细，最灵活以及学习起来最困难的框架，比如配置个路由都可以写很长的数组结构，以及各模块的耦合性及复杂度，还有一大堆没人使用的模块等，其后果是显而易见的，对框架关注及使用的用户根本未达到zend官方的期待,在不就的将来zf2必死,zf2第三方模块(modules.zendframework.com)已经很久没有新的出现了

### 为什么使用Zend Framework 3

1.在ZF2中有几种设计错误，解决办法通过创建一个新的版本 2.模块性能的优化，5-10倍 3.ZF2组件都具有复杂的依赖关系，很难分离成一个个单个组建 4.一些组件过于复杂化，中间无用的代码过多，需要重构其功能

### Zend Framework 3 与 Zend Framework 2 区别

没有多少区别，ZF2有正确的抽象，ZF3的目的只是为了让他们更好，更高效。 同时还会有很多重大更改的，ZF3的架构将是非常接近ZF2。 它绝对不会像ZF1到ZF2。在ZF2中很多组件几乎没多少人使用(rbac,很少人有人用原生的rbac组件,就算用也要重构),有些做api的或者用前端js框架的(比如angularjs..),form,paginator等组件几乎用不到，所以在ZF3的组件会区分主要的和次要的及第三方组件，目标是让组件更独立 

主体框架的组件列表

Authentication

Crypt

Escaper

EventManager

Filter

Http

I18n

InputFilter

Validator

Log

ModuleManager

Router

Mvc

ServiceManager

View

提供其他的扩展组件

Feed

Form

Paginator

Session

第三方组件

RBAC
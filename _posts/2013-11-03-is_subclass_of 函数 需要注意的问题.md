---
layout: post
title: "is_subclass_of 函数 需要注意的问题"
date: 2013-11-03
categories: php
---

```php
#判断$className是否是$type的子类
is_subclass_of($className,$type);
```

php5.3.7版本前针对interface会有一个bug

bug:https://bugs.php.net/bug.php?id=53727

```php
interface MyInterface {}
class ParentClass implements MyInterface { }
class ChildClass extends ParentClass { }

# true
is_subclass_of('ChildClass', 'MyInterface');
# false
is_subclass_of('ParentClass', 'MyInterface');
[/php]

解决办法

```php
function isSubclassOf($className, $type){
    // 如果 $className 所属类是 $type 的子类，则返回 TRUE
    if (is_subclass_of($className, $type)) {
        return true;
    }

    // 如果php版本>=5.3.7 不存在interface bug 所以 $className 不是 $type 的子类
    if (version_compare(PHP_VERSION, '5.3.7', '>=')) {
        return false;
    }

    // 如果$type不是接口 也不会有bug 所以 $className 不是 $type 的子类
    if (!interface_exists($type)) {
        return false;
    }

    //  创建一个反射对象
    $r = new ReflectionClass($className);
    //  通过反射对象判断该类是否属于$type接口
    return $r->implementsInterface($type);
}
```
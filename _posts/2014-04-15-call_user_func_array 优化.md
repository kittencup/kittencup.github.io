---
layout: post
title: "call_user_func_array 优化"
date: 2014-04-15
categories: php
---

在PHP里，如果想动态调用一个函数，可以写成这样

```php

$string = '小猫杯,Kittencup';

$funName = 'explode';

var_dump($funName(',',$string));

var_dump(call_user_func_array($funName,array(',',$string)));

```

在一些成熟的框架中，动态调用一个不知参数个数的函数时，代码会写成这样

```php

class Kittencup
{

    protected $webUrl = 'http://%s.kittencup.com';

    public function getWebUrl($name)
    {
        return sprintf($this->webUrl, $name);
    }

    public function __call($funName, $args)
    {
        switch (count($args)) {
            case 0:
                return $this->$funName();
            case 1:
                return $this->$funName($args[0]);
            case 2:
                return $this->$funName($args[0], $args[1]);
            case 3:
                return $this->$funName($args[0], $args[1], $args[2]);
            case 4:
                return $this->$funName($args[0], $args[1], $args[2], $args[3]);
            case 5:
                return $this->$funName($args[0], $args[1], $args[2], $args[3], $args[4]);
            default:
                return call_user_func_array($this->$funName, $args);
        }
    }
}

$kittencup = new Kittencup();
echo $kittencup->getWebUrl('www');

```

call_user_func_array 通常会有一个额外的函数调用的开销，当然这个开销是在微秒的范围内，但它可以在两种情况下成为重要的： 递归函数调用 : 由于call_user_func_array的加入再次调用堆栈，它会增加一倍堆栈的使用量，使用上面代码__call这种写法可以在递归调用时减少多达33％堆栈的使用 多次调用 : 如果你调用的函数很多，一般这种写法会在一些大型OOP框架内，比如Zend Framework 2等，在应用程序的生命周期，它可能会被调用几十，几百甚至几千倍，即使每个单独的消耗是微秒，累积起来也会有一定量的性能消耗。
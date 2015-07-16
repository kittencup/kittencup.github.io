---
layout: post
title: "合理使用PHP SPL Exception"
date: 2015-02-14
categories: php
---

PHP的SPL添加很多Exception,在项目中需要合理的使用，主要分为两大类LogicException和RuntimeException

```php
LogicException (extends Exception)
	BadFunctionCallException
		BadMethodCallException
	DomainException
	InvalidArgumentException
	LengthException
	OutOfRangeException
RuntimeException (extends Exception)
	OutOfBoundsException
	OverflowException
	RangeException
	UnderflowException
	UnexpectedValueException

```

2.LogicException与RuntimeException的区别
--------
RuntimeExcption是指程序本身以外无法由开发者控制的状况，比如访问远程API,API却没有传回正确的内容，CURL抓取外部的页面，页面不存在，或者数据库查询(PdoException就是继承了RuntimeException);

LogicException是指程序本身的问题，应该是开发者事前就解决的问题，比如class未被引入，调用了不存在方法等

一个正常的程序，应该可以允许RuntimeException的出现，但是LogicException应该完全看不到

3.LogicException
--------
* BadFunctionCallException 在调用一个未定义的，或不允许被使用的函数，或者使用is_callable()返回false时候使用，这个是开发者决定问题，故属于LogicException

```php
if (!is_callable($function))
{
    throw new BadFunctionCallException('Function is not callable.');
}
```

* BadMethodCallException，类似于BadFunctionCallException，是Method用的版本，有些特殊的对象(Immutable Design object)，不能调用set或者construct时，也可以使用该异常

```php
namespace Kittencup\Uri;

final class UriImmutable extends AbstractUri
{
	protected $constructed = false;
	
	public function __set($name, $value)
	{
		throw new \BadMethodCallException('这个是一个immutable对象');
	}
	
	/**
	 * 只有初始化时能设置该对象，第2次在调用__construct方法时，则抛出异常
	 */
	public function __construct($uri = null)
	{
		if ($this->constructed === true)
		{
			throw new \BadMethodCallException('这个是一个immutable对象');
		}
		$this->constructed = true;
		parent::__construct($uri);
	}
}
```

* DomainException,调用的内容不在预定义的范围之内，比如有时想调用一个第3方类库，类库还不存在autoload中，可以抛出该异常，
或者解析一张图片，但是图片格式又不在你能解析的范围内

```php
switch ($type) {
    case 'png':
        // ...
        break;
    case 'jpeg':
        // ...
        break;
    case 'gif':
        // ...
     break;
    default:
        throw new \DomainException('未知的图片类型');
}		
```
* InvalidArgumentException 不符合规定的参数时抛出异常

```php
function kittencup($arrayOrString)
{
    if (!is_array($arrayOrString) || !is_string($arrayOrString))
    {
        throw new InvalidArgumentException('参数必须是数组或对象');
    }
}
```

* LengthException 长度不符合的时候抛出该异常

```php
public static function randomElements(array $array = array('a', 'b', 'c'), $count = 1)
    {
        $allKeys = array_keys($array);
        $numKeys = count($allKeys);
        if ($numKeys < $count) {
            throw new LengthException('数组元素内容不能少于随机抽取的数量');
        }
        $highKey = $numKeys - 1;
        $keys = $elements = array();
        $numElements = 0;
        while ($numElements < $count) {
            $num = mt_rand(0, $highKey);
            if (isset($keys[$num])) {
                continue;
            }
            $keys[$num] = true;
            $elements[] = $array[$allKeys[$num]];
            $numElements++;
        }
        return $elements;
    }
```

* OutOfRangeException 试图从已知长度的集合中存取不合法或不在该范围的内容时或者在一些数字型参数不符合正常思维时，抛出该异常

```php
namespaces Kittencup

class Student{
	
	protected $students;
	
	public function getStudentByIndex($index)
	{
	    if ($index < 0 || $index >= count($this->students)) {
	        throw new Exception\OutOfRangeException('Index 超出了总共的范围');
	    }
	    return $this->students[$index];
	}
	
	public function getRangeStudent($min,$max){
		 if ($min > $max) {
            throw new Exception\OutOfRangeException('max必须大于min');
         	}
		// ....
	}
}


```

4.RuntimeException
--------

* OutOfBoundsException 当从未知长度的集合中获取不合法的索引值数据时，因为不确定长度意味着数据是外部资源决定的(database等)，不属于开发者控制范围，请和Logic的OutOfRangeException合理的区分

```php
$user = User::findAll();

if(!isset($user[100])){
	 throw new OutOfBoundsException('用户 ' . $i . ' 不存在.');
}
```
* OverflowException 当加入一个元素到满的集合时，且这个集合不被开发者控制的时候，比如用作内存缓存时，添加数据需要检测下剩余内存大小

```php
// zf2 的Memory Cache
protected function internalSetItem(& $normalizedKey, & $value)
{
    $options = $this->getOptions();

    if (!$this->hasAvailableSpace()) {
        $memoryLimit = $options->getMemoryLimit();
        throw new Exception\OutOfSpaceException(
            "Memory usage exceeds limit ({$memoryLimit})."
        );
    }

    $ns = $options->getNamespace();
    $this->data[$ns][$normalizedKey] = array($value, microtime(true));

    return true;
}	
```

* RangeException 某个extension的未安装，extension不一定是开发者可以控制的，可能是主机商决定，请和Logic的DomainException合理的区分

```php
if (!extension_loaded('mysqli'))
{
    throw new RangeException('Mysqli未被安装');
}
```

* UnderflowException 在一个空容器进行无效操作的时候，比如移除元素抛出的异常。

```php
function removeElement($array){
	if (count($array) > 1){
	    array_pop($array);
	}else{
	    throw new UnderflowException('没有元素可以被移除了.');
	}
}
```

* UnexpectedValueException 返回的结果不是预期的值或类型时，例如数据库，curl,打开一个文件等可以抛出异常

```php
$fileHandle = fopen($filename, 'r');
if (false === $fileHandle) {
    throw new UnexpectedValueException("不能读取$filename");
}
```

5.开发时的建议
--------

开发一个模块(Module,Bundle,Package)时，建立独立的Exception，在捕获时可以区分不同模块的异常，方便进行不同的设置

```php
// 一个标准的psr-0模块

Kittencup
	config
	src
		Kittencup
			Controller
			Exception
				ExceptionInterface
				UnexpectedValueException
				......
			....
			
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

// ExceptionInterface.php
namespace Kittencup\Exception;

interface ExceptionInterface
{
}
			
// UnexpectedValueException.php
namespace Kittencup\Exception;

class UnexpectedValueException extends \UnexpectedValueException implements ExceptionInterface
{
}

// 抛出异常时候
throw new Kittencup\Exception\UnexpectedValueException

// 捕获异常
try{

}catch(\Kittencup\Exception\UnexpectedValueException $e){
    // 这个是你抛出的，万一try里其他模块也抛出UnexpectedValueException异常，这样就可以区分不同的异常不同处理
}catch(\Zend\Authentication\Adapter\Exception\UnexpectedValueException $e){

}

```






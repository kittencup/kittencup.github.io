---
layout: post
title: "2015-02-18-Laravel 5 Eloquent 属性 Casts"
date: 2015-02-18
categories: php
---

Laravel 5 Eloquent 提供了对属性的类型进行自动转换

2.布尔值的问题
--------
有时候我们在进行数据库设计的时候，往往会使用数字0和1代表开启和关闭，在通过Eloquent查询出来后我们会得到“1”或“0”的字符串，
幸运的是，PHP会将其转换为正确的布尔值，但是如果你通过JSON转换来返回给javascript来处理，那么结果就不同了

```
<?php
$category = [
    'name' => 'kittencup',
    'enabled' => "0"
];

echo $category['enabled'] ? '开启' : '关闭'; // 将会输出关闭

?>


<script>

    var categoryJson = '<?php echo json_encode($category);?>'

    var category = JSON.parse(categoryJson);

    alert(category.enabled ? '开启' : '关闭');  // 将会输出开启
</script>

```
所以有时候你不得不在php或者js中，强制手动转换下数据类型，然后在进行判断


3.laravel 5 Eloquent 提供的$casts属性来解决这个问题
--------

在model中定义$casts属性

```
<?php namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
	......
	
	protected $casts = [
		'enabled' => 'integer'
	];
    ......
}

```
Eloquent 会根据你的$casts设置，对结果值进行转换，从Model源代码中获取支持的转换的类型

```
/**
	 * Cast an attribute to a native PHP type.
	 *
	 * @param  string  $key
	 * @param  mixed   $value
	 * @return mixed
	 */
	protected function castAttribute($key, $value)
	{
		switch ($this->getCastType($key))
		{
			case 'int':
			case 'integer':
				return (int) $value;
			case 'real':
			case 'float':
			case 'double':
				return (float) $value;
			case 'string':
				return (string) $value;
			case 'bool':
			case 'boolean':
				return (bool) $value;
			case 'object':
				return json_decode($value);
			case 'array':
			case 'json':
				return json_decode($value, true);
			default:
				return $value;
		}
	}
```

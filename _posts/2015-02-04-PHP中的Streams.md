---
layout: post
title: "PHP中的Streams"
date: 2015-02-04
categories: php
---

Streams 是PHP提供的一个强有力的工具，如果善加利用将大大提高PHP的生产力，如今在psr-7中的http message 主体中 也用到了Stream功能 PHP手册中对Streams的一段描述：

> Streams 是在PHP 4.3.0版本被引入的，它被用于统一文件、网络、数据压缩等类文件的操作方式，为这些类文件操作提供了一组通用的函数接口。简而言之，一个stream就是一个具有流式行为的资源对象。也就是说，我们可以用线性的方式来对stream进行读取和写入。并且可以用使用fseek()来跳转到stream内的任意位置。

每个Streams对象都有一个包装类，在包装中可以添加处理特殊协议和编码的相关代码。PHP中已经内置了一些常用的包装类，我们也可以创建和注册自定义的包装类。我们甚至能够使用现有的context和filter对包装类进行修改和增强。

### Stream 基础知识

Stream 可以通过<scheme>://<target>方式来引用。其中<scheme>是包装类的名字，<target>中的内容是由包装类的语法指定，不同的包装类的语法会有所不同。 PHP默认的包装类是file://，也就是说我们在访问文件系统的时候，其实就是在使用一个stream。我们可以通过下面两种方式来读取文件中的内容，readfile('/path/to/somefile.txt')或者readfile('file:///path/to/somefile.txt')，这两种方式是等效的。如果你是使用readfile('http://google.com/')，那么PHP会选取HTTP stream包装类来进行操作。 查看本地PHP内置支持的协议和封装协议(PHP 5.5.20)

```php
var_dump(stream_get_wrappers());

//array (size=12)
//  0 => string 'https' (length=5)
//  1 => string 'ftps' (length=4)
//  2 => string 'compress.zlib' (length=13)
//  3 => string 'compress.bzip2' (length=14)
//  4 => string 'php' (length=3)
//  5 => string 'file' (length=4)
//  6 => string 'glob' (length=4)
//  7 => string 'data' (length=4)
//  8 => string 'http' (length=4)
//  9 => string 'ftp' (length=3)
//  10 => string 'phar' (length=4)
//  11 => string 'zip' (length=3)
```

file:// 访问本地文件系统
file://是PHP使用的默认封装协议，展现了本地文件系统

```php
file_get_contents('/Users/Kittencup/WebServer/test/001.php');
file_get_contents('file:///Users/Kittencup/WebServer/test/001.php');
```

php:// 访问各个输入/输出流（I/O streams）
php://stdin
允许直接访问PHP进程相应的输入流

```php
#stdin.php
$stdin = fopen ("php://stdin","r");
$line = fgets($stdin);
fclose($stdin);
echo '你输入的是 ' . $line;
```

使用cli执行

```php
php stdin.php
kittencup
你输入的是 kittencup
```

推荐使用常量代替打开这些封装器

```php
# stdin.php
$line = fgets(STDIN);
echo '你输入的是 ' . $line;
fclose($stdin);
```

php://stdout
允许直接访问PHP进程相应的输出流

```php
#stdout.php
// fwrite(fopen('php://output', 'w'), "Kittencup\n");
fwrite(fopen(STDOUT, 'w'), "Kittencup\n");
```

```bash
php stdout.php
Kittencup
```

output会绕过ob
```php
#stdout.php
ob_start();
echo "ob_contents\n";
fwrite(STDOUT, "Kittencup\n");
$ob_contents = ob_get_clean();
echo $ob_contents;
```

```bash
php stdout.php
Kittencup
ob_contents
```

php://stderr
和stdout类似 输出到错误通道,stdout是标准输出流，默认为屏幕,
stderr是标准错误流，一般把屏幕设为默认,可以根据情况分别降stderr和stdout重定位到其它设备上，以区别对待正常输出和错误报告。

php://input
访问请求的原始数据的只读流

```bash
curl -d "Kittencup" -d "a=3&b=4" http://test.kittencup.com/001.php
```

```php
var_dump($_POST)
```

会丢失第一个Kittencup

```php
echo file_get_contents('php://input');
// Kittencup&a=3&b=4
```

php://output
只写的数据流，以 print 和 echo 一样的方式写入到输出区
php://memory 和 php://temp
php://memory 和 php://temp 是一个类似文件包装器的数据流，允许读写临时数据。
两者的唯一区别是 php://memory 总是把数据储存在内存中， 而php://temp 会在内存量达到预定义的限制后（默认是 2MB）存入临时文件中。
php://fd
允许直接访问指定的文件描述符
0代表stdin,1代表stdout,2代码stderr,其他的默认从3开始

```php
fopen('php://fd/0','r')
```

php://filter
在使用 readfile()，file_get_contents()，stream_get_contents()之类的函数使，可以筛选过滤应用

```php
var_dump(stream_get_filters());
//array (size=12)
//  0 => string 'zlib.*' (length=6)
//  1 => string 'bzip2.*' (length=7)
//  2 => string 'convert.iconv.*' (length=15)
//  3 => string 'string.rot13' (length=12)
//  4 => string 'string.toupper' (length=14)
//  5 => string 'string.tolower' (length=14)
//  6 => string 'string.strip_tags' (length=17)
//  7 => string 'convert.*' (length=9)
//  8 => string 'consumed' (length=8)
//  9 => string 'dechunk' (length=7)
//  10 => string 'mcrypt.*' (length=8)
//  11 => string 'mdecrypt.*' (length=10)

echo file_get_contents("php://filter/read=string.toupper/resource=file:///Users/kittencup/WebServer/test/001.php");
```

更多具体的使用方法参考手册

其他的协议

http:// — 访问 HTTP(s) 网址

ftp:// — 访问 FTP(s) URLs

zlib:// — 压缩流

data:// — 数据（RFC 2397）

glob:// — 查找匹配的文件路径模式

phar:// — PHP 归档

ssh2:// — Secure Shell 2

rar:// — RAR

ogg:// — 音频流

expect:// — 处理交互式的流

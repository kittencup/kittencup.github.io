---
layout: post
title: "使用PHP解析PDF文件之 (二) PDF验证及获取交叉应用表的位置"
date: 2015-02-12
categories: php
---

本文主要讲解使用PHP，验证是否是PDF文件，获取交叉应用表的位置

2.将PDF文件读取到内存中
--------
```php
$content = file_get_contents('test.pdf');
```
3.验证PDF文件
-------- 
PDF文件必带PDF规定的规范的版本号，PDF的具体内从则从PDF版本号开始

```php
$position = strpos($content, '%PDF-');
if ($position === false) {
   // 不是pdf文件
}

// PDF的内容是从版本号开始，版本号之可能还有其他无用数据
$content = substr($content, $position);
```


4.获取交叉应用表的位置
-------- 
```bash
%PDF-1.3 %Äåòåë§ó ÐÄÆ 4 0 obj << /Length 5 0 R /Filter /FlateDecode >> stream xM Â0…÷=Åwí$mjâB
........
endobj xref 0 75 0000000000 65535 f 0000230284 00000 n 0000000253 00000 n 0000214387 00000 n 0000000022 00000 n 0000000234 00000 n 0000000358 00000 n 0000004450 00000 n 0000003201 00000 n 0000214556 00000 n 0000000465 00000 n 0000003180 00000 n 0000003237 00000 n 0000004429 00000 n 0000004858 00000 n
........
trailer << /Size 75 /Root 50 0 R /Info 1 0 R /ID [ ] >> startxref 230428 %%EOF
```

通过在源码中找到最后的startxref来获取xref的相对偏移位置230428

```php
if (!preg_match_all('/[\r\n]startxref[\s]*[\r\n]+([0-9]+)[\s]*[\r\n]+%%EOF/i', $content, $matches, PREG_SET_ORDER)) {
  	// 找不到startxref,表示pdf有问题
}

$matches = array_pop($matches);
$xrefOffset = $matches[1];
// $xrefOffset = 230428;
```

PHP就可以通过23048找到xref开始的位置



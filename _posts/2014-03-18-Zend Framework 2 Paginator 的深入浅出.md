---
layout: post
title: "Zend Framework 2 Paginator 的深入浅出"
date: 2014-03-18
categories: php
---

### Zend\Paginator 是什么，能做什么？

Paginator 顾名思义，分页程序，大部分人用作数据库数据的分页操作，Zend Framwork 2 的分页不仅仅可以对数据库数据进行分页，还能对数组，迭代器进行分页，还支持对分页的数据缓存，过滤

### Zend\Paginator 模块的文件组成

主要分适配器(Adpater),分页样式(ScrollingStyle),分页样式管理器(ScrollingStylePluginManager),主分页程序(Paginator)

#### 适配器(Adpater):

> 提供分页数据，使用适配器设计模式，Adapter主要保存要分页的数据总数，及适配器统一接口getItems()来获取对应的分页数据

#### 分页列表样式(ScrollingStyle):

> 中间分页列表的算法实现

#### 分页样式管理器(ScrollingStylePluginManager):

> 主分页程序(Paginator)通过ScrollingStylePluginManager来生成对应的ScrollingStyle对象

#### 主分页程序(Paginator):

> Paginator实现了Countable和IteratorAggregate接口,当count(Paginator)获取的是一共分几页，当迭代Paginator时，会通过Adpater里的getItems 来获取当前页的数据

#### Paginator.php类定义

```php

class Paginator implements Countable, IteratorAggregate
{
    // 缓存标签前缀
    const CACHE_TAG_PREFIX = 'Zend_Paginator_';
    // 中间分页列表管理器适配器对象
    protected static $adapters = null;
    // 全局配置
    protected static $config = null;
    // 默认中间分页列表算法
    protected static $defaultScrollingStyle = 'Sliding';
    // 默认每页显示几条
    protected static $defaultItemCountPerPage = 10;
    // 中间分页列表管理器对象
    protected static $scrollingStyles = null;
    // 缓存适配器对象
    protected static $cache;
    // 是否开启缓存
    protected $cacheEnabled = true;
    // 适配器对象
    protected $adapter = null;
    // 当前页数据总数
    protected $currentItemCount = null;
    // 当前页数据
    protected $currentItems = null;
    // 当前页数
    protected $currentPageNumber = 1;
    // 过滤器对象
    protected $filter = null;
    // 每页显示几条
    protected $itemCountPerPage = null;
    // 总分页数
    protected $pageCount = null;
    // 中间分页列表算法范围
    protected $pageRange = 10;
    // 分页的详细数据对象
    protected $pages = null;
    // 渲染视图的对象
    protected $view = null;

    // 初始化
    public function __construct($adapter){}
    // 设置全局配置
    public static function setGlobalConfig($config){}
    // 获取默认的中间分页列表样式算法
    public static function getDefaultScrollingStyle(){}
    // 设置默认中间分页列表样式算法
    public static function setDefaultScrollingStyle($scrollingStyle = 'Sliding'){}
    // 获取默认每页显示几条
    public static function getDefaultItemCountPerPage(){}
    // 设置默认每页显示几条
    public static function setDefaultItemCountPerPage($count){}
    // 设置缓存对象
    public static function setCache(CacheStorage $cache){}
    // 设置中间分页列表管理器
    public static function setScrollingStylePluginManager($scrollingAdapters){}
    // 获取中间分页列表管理器
    public static function getScrollingStylePluginManager(){}
    // 将分页渲染到view
    public function __toString(){}
    // 设置缓存是否开启
    public function setCacheEnabled($enable){}
    // 计算总共分几页
    public function count(){}
    // 获取数据总数
    public function getTotalItemCount(){}
    // 清除某页或全部分页数据缓存
    public function clearPageItemCache($pageNumber = null){}
    // 获取数据所在页数
    public function getAbsoluteItemNumber($relativeItemNumber, $pageNumber = null){}
    // 获取数据适配器
    public function getAdapter(){}
    // 获取当前页的数据总数
    public function getCurrentItemCount(){}
    // 获取当前页的数据
    public function getCurrentItems(){}
    // 获取当前页数
    public function getCurrentPageNumber(){}
    // 设置当前页数
    public function setCurrentPageNumber($pageNumber){}
    // 获取过滤器
    public function getFilter(){}
    // 设置过滤器
    public function setFilter(FilterInterface $filter){}
    // 根据数据所在位置或者页数 获取数据
    public function getItem($itemNumber, $pageNumber = null){}
    // 获取每页显示几条
    public function getItemCountPerPage(){}
    // 设置每页显示几条  -1 为显示所有数据
    public function setItemCountPerPage($itemCountPerPage = -1){}
    // 根据数据 计算数据条数
    public function getItemCount($items){}
    // 根据页数获取数据
    public function getItemsByPage($pageNumber){}
    // 获取当前页的数据迭代器
    public function getIterator(){}
    // 获取中间分页列表算法范围
    public function getPageRange(){}
    // 设置中间分页列表算法范围
    public function setPageRange($pageRange){}
    // 获取当前分页详细数据
    public function getPages($scrollingStyle = null){}
    // 根据最低边界，最高边界，获取分页列表的数据
    public function getPagesInRange($lowerBound, $upperBound){}
    // 获取分页缓存数据
    public function getPageItemCache(){}
    // 获取视图
    public function getView(){}
    // 设置视图
    public function setView(View\Renderer\RendererInterface $view = null){}
    // 标准化数据位置编号
    public function normalizeItemNumber($itemNumber){}
    // 标准化页数
    public function normalizePageNumber($pageNumber){}
    // 渲染到view里
    public function render(View\Renderer\RendererInterface $view = null){}
    // 分页数据json
    public function toJson(){}
    // 检查是否开启缓存
    protected function cacheEnabled(){}
    // 获取缓存ID
    protected function _getCacheId($page = null){}
    // 获取内部缓存ID
    protected function _getCacheInternalId(){}
    // 计算分页总数
    protected function _calculatePageCount(){}
    // 创建分页详细数据
    protected function _createPages($scrollingStyle = null){}
    // 加载中间分页列表对象
    protected function _loadScrollingStyle($scrollingStyle = null){}
}

```

### 简单的使用示例

```php
include 'init_autoloader.php';

$data = array('a','b','c','d','e','f','g','h','i','j','k','m','n','o','p','q','r','s','t','u','v','w','x','y','z');

// 创建一个数组数据适配器
$adapter = new Zend\Paginator\Adapter\ArrayAdapter($data);

// 创建主的分页程序
$paginator = new Zend\Paginator\Paginator($adapter);

// 迭代当前页面的分页数据
foreach ($paginator as $item) {
    echo $item, '<br/>';
}

// 获取分页条的详细数据
$pages = $paginator->getPages();

?>
<!--生成分页HTML-->
<ul>
    <li><a href="<?= $pages->first ?>">第一页</a></li>

    <?php if (isset($pages->previous)): ?>
        <li><a href="<?= $pages->previous ?>">上一页</a></li>
    <?php endif; ?>

    <?php if (isset($pages->pagesInRange)): ?>
        <?php foreach ($pages->pagesInRange as $page): ?>
            <li><a href="<?= $page ?>"><?= $page ?></a></li>
        <?php endforeach; ?>
    <?php endif; ?>

    <?php if (isset($pages->next)): ?>
        <li><a href="<?= $pages->next ?>">下一页</a></li>
    <?php endif; ?>

    <li><a href="<?= $pages->last ?>">最后一页</a></li>
</ul>

```

### Paginator 常用的配置

```php

// 设置的当前页数，每页显示几条，分页中间样式算法
$paginator->setCurrentPageNumber(1)->setItemCountPerPage(1)->setDefaultScrollingStyle('All');

```

### 全局配置

在一个站里，需要分页的地方有很多，Paginator提供了一个全局配置的静态属性，简化我们对每一个分页的设置 配置参考:

```php

$config = array(
    // ScrollingStylePluginManager设置 类名或者对象
    'scrolling_style_plugins'=>'Zend\Paginator\ScrollingStylePluginManager',
    // 中间分页样式算法 设置
    'scrolling_style'=>'All',
    // 设置每页显示几条
    'itemcountperpage'=>3,
    // 设置中间列表算法的范围值
    'pagerange'=>5
);

// 注册进Paginator
Zend\Paginator\Paginator::setGlobalConfig($config);

```

### 数据缓存

Paginator里的缓存，仅用在一次HTTP请求上多次请求分页数据，因为每次页面刷新后缓存ID就会变化，所以Paginator里的缓存不能用作于持久缓存 缓存示例:

```php

$data = array('a','b','c','d','e','f','g','h','i','j','k','m','n','o','p','q','r','s','t','u','v','w','x','y','z');

// 创建一个数组数据适配器
$adapter = new Zend\Paginator\Adapter\ArrayAdapter($data);

// 创建主的分页程序
$paginator = new Zend\Paginator\Paginator($adapter);

// 创建cacheStorage
$cacheStorage = Zend\Cache\StorageFactory::factory(array(
                                                        'adapter' => 'filesystem',
                                                        'options' => array(
                                                            'cache_dir' => 'data/cache/paginator',
                                                        ),
                                                        'plugins' => array('serializer'),
                                                   ));
// paginator设置缓存
Zend\Paginator\Paginator::setCache($cacheStorage);
// paginator开启缓存
$paginator->setCacheEnabled(true);
// 获取第一页数据 无缓存 会创建缓存
$data1 = $paginator->getItemsByPage(1);
var_dump($data1);
// 再一次获取第一页数据 有缓存 直接返回缓存
$data2 = $paginator->getItemsByPage(1);
var_dump($data2);
// 清除第一页缓存
$paginator->clearPageItemCache(1);
// 清除所有页面缓存
$paginator->clearPageItemCache();

```

### 过滤器

Paginator也可配合Zend\Filter 过滤分页数据 过滤示例:

```php

$data = array('a','b','c','d','e','f','g','h','i','j','k','m','n','o','p','q','r','s','t','u','v','w','x','y','z');

// 创建一个数组数据适配器
$adapter = new Zend\Paginator\Adapter\ArrayAdapter($data);

// 创建主的分页程序
$paginator = new Zend\Paginator\Paginator($adapter);

// 使用Zend\Filter\Callback 定义一个过滤器 将分页数据设置为大写
$filter = new Zend\Filter\Callback(function ($data) {
    return array_map('strtoupper', $data);
});
// 为Paginator设置过滤器
$paginator->setFilter($filter);

// 迭代当前页面的分页数据
foreach ($paginator as $item) {
    echo $item, '';
}

```
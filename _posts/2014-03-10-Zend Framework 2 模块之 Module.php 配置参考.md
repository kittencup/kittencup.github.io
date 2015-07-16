---
layout: post
title: "Zend Framework 2 模块之 Module.php 配置参考"
date: 2014-03-10
categories: php
---

```php
<?php
namespace Blog;

use Zend\ModuleManager\Feature\ConfigProviderInterface;
use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
use Zend\ModuleManager\Feature\DependencyIndicatorInterface;
use Zend\ModuleManager\Feature\InitProviderInterface;
use Zend\ModuleManager\Feature\LocatorRegisteredInterface;

use Zend\ModuleManager\Feature\ServiceProviderInterface;
use Zend\ModuleManager\Feature\ControllerProviderInterface;
use Zend\ModuleManager\Feature\ControllerPluginProviderInterface;
use Zend\ModuleManager\Feature\ViewHelperProviderInterface;
use Zend\ModuleManager\Feature\ValidatorProviderInterface;
use Zend\ModuleManager\Feature\FilterProviderInterface;
use Zend\ModuleManager\Feature\FormElementProviderInterface;
use Zend\ModuleManager\Feature\RouteProviderInterface;
use Zend\ModuleManager\Feature\SerializerProviderInterface;
use Zend\ModuleManager\Feature\HydratorProviderInterface;
use Zend\ModuleManager\Feature\InputFilterProviderInterface;

use Zend\ModuleManager\Feature\BootstrapListenerInterface;

use Zend\ModuleManager\ModuleManagerInterface;
use Zend\EventManager\EventInterface;

class Module implements AutoloaderProviderInterface,
                        DependencyIndicatorInterface,
                        InitProviderInterface,
                        LocatorRegisteredInterface,
                        ConfigProviderInterface,

                        ServiceProviderInterface,
                        ControllerProviderInterface,
                        ControllerPluginProviderInterface,
                        ViewHelperProviderInterface,
                        ValidatorProviderInterface,
                        FilterProviderInterface,
                        FormElementProviderInterface,
                        RouteProviderInterface,
                        SerializerProviderInterface,
                        HydratorProviderInterface,
                        InputFilterProviderInterface,

                        BootstrapListenerInterface
{
    /**
     * 当loadMoule(9000)事件时，会触发
     * Zend\ModuleManager\Listener\AutoloaderListener
     * __invoke()方法 通过getAutoloaderConfig里返回的数组配置
     * 传递给AutoloaderFactory::factory()，注册自动加载机制
     *
     * 对应的interface =&gt; AutoloaderProviderInterface
     * @return array
     */
    public function getAutoloaderConfig()
    {
        return array(
            'Zend\Loader\StandardAutoloader' =&gt; array(
                'namespaces' =&gt; array(
                    __NAMESPACE__ =&gt; __DIR__ . '/src/' . __NAMESPACE__,
                ),
            ),
        );
    }

    /**
     * 当loadModule(8000)事件时，会触发
     * Zend\ModuleManager\Listener\ModuleDependencyCheckerListener
     * __invoke()方法 会调用getModuleDependencies()方法，方法里返回的模块必须在当前模块之前加载
     *
     * 需要检查依赖的前提必须在
     * application.config.php里的module_listener_options配置里的键值check_dependencies设置为true或不设置(默认为真)
     * 设置 check_dependencies=&gt;false 可以关闭注册依赖检查事件
     * 对应的interface =&gt; DependencyIndicatorInterface
     * @return array
     */
    public function getModuleDependencies()
    {
        return array(
            'Application'
        );
    }

    /**
     * 当loadModule事件时，会触发
     * Zend\ModuleManager\Listener\InitTrigger
     * __invoke()方法 会调用每个模块的init()方法，并将 Zend\ModuleManager\ModuleManager 注入进来
     * 对应的interface =&gt; InitProviderInterface
     */
    public function init(ModuleManagerInterface $manager)
    {

    }

    /**
     * 当loadModule事件时，会触发
     * Zend\ModuleManager\Listener\ConfigListener
     * onLoadModule()方法，会调用模块里 getConfig()方法，方法需要返回当前模块的配置
     * 对应的interface =&gt; ConfigProviderInterface
     * @return array|mixed|\Traversable
     */
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }

    /**
     * 当loadModule事件时，会触发
     * Zend\ModuleManager\Listener\ServiceListener
     * onLoadModule() 方法调用这些方法，获取当前这些服务配置，会分别将对应的配置注册进服务
     * 这些配置也可在getConfig()方法返回的配置内容中配置，最后都会合并起来
     * 对应的interface =&gt;
     * Zend\ModuleManager\Feature\ServiceProviderInterface;
     * Zend\ModuleManager\Feature\ControllerProviderInterface;
     * Zend\ModuleManager\Feature\ControllerPluginProviderInterface;
     * Zend\ModuleManager\Feature\ViewHelperProviderInterface;
     * Zend\ModuleManager\Feature\ValidatorProviderInterface;
     * Zend\ModuleManager\Feature\FilterProviderInterface;
     * Zend\ModuleManager\Feature\FormElementProviderInterface;
     * Zend\ModuleManager\Feature\RouteProviderInterface;
     * Zend\ModuleManager\Feature\SerializerProviderInterface;
     * Zend\ModuleManager\Feature\HydratorProviderInterface;
     * Zend\ModuleManager\Feature\InputFilterProviderInterface;
     *
     */
    public function getServiceConfig(){}

    public function getControllerConfig(){
        return array();
    }

    public function getControllerPluginConfig(){}

    public function getViewHelperConfig(){}

    public function getValidatorConfig(){}

    public function getFilterConfig(){}

    public function getFormElementConfig(){}

    public function getRouteConfig(){}

    public function getSerializerConfig(){}

    public function getHydratorConfig(){}

    public function getInputFilterConfig(){}

    /**
     * 在bootstrap事件时触发,触发时会传入 Zend\EventManager\Event对象
     * 该注册事件是在loadModule事件时，通过
     * Zend\ModuleManager\Listener\OnBootstrapListener
     * invoke()方法里 如果当前模块有onBootstrap()方法,则
     * 在sharedEventManager上注册事件bootstrap，id为Zend\Mvc\Application
     */
    public function onBootstrap(EventInterface $e){}

}

```
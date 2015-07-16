---
layout: post
title: "Zend Framework 3 BC-breaks"
date: 2015-02-03
categories: php
---

BC-breaks => Backwards Compatibility Breaks => 向后兼容性的破坏

### Input Filters

*   InputFilter组件重写，提升性能及简洁的代码
*   InputFilter改名为InputCollection以更好的地反映其目的
*   InputCollection现在只是扩展输入，这样它们就可以附加验证器和过滤器
*   验证组现在已经完全从InputCollection脱离出来
*   举例来说，如果你想创建一个数组验证组：（zf2是默认行为，不需要此代码）

[php] $inputCollection = new InputCollection(); $validationGroup = new ArrayFilterIterator($inputCollection, ['field1', 'field2']); $inputCollection->setValidationGroupFilter($validationGroup); [/php] 虽然代码需要多打一些，但更高效，灵活。ZF3自带的几个内置的验证组过滤迭代器：NoOpFilterIterator，ArrayFilterIterator，CallableFilterIterator和RegexFilterIterator

*   Input 和 InputCollection 现在完全是无状态的，这有几方面含义，首先，在相同的InputCollection或Input可以多次重复使用，因此减少了需要具有共享的实例，验证结果和过滤值则在 "InputFilterResult"对象内，给其定义了isValid，getRawData，和的getData方法getErrorMessages方法

[php] // ... create the input collection $inputFilterResult = $inputCollection->runAgainst(['field1' => 'value1']); if ($inputFilterResult->isValid()) { $values = $inputFilterResult->getData(); // 或者使用 getRawData(); } else { $error = $inputFilterResult->getErrorMessages(); } [/php] InputFilterResult是可序列化，也实现PHP 5.4 JsonSerializable接口，所以你可以json_encode($inputFilterResult)，它会自动序列化的错误消息。

### Filters(过滤器)

*   所有的filters现在可当函数调用方法(实现__invoke())。
*   各种选项(options)现在使用下划线分离(underscore_separated)，像框架的其他部分。
*   一些filters被改写，以减少代码和提高性能。
*   Tar和Gz filters：方法“setMode”/“getMode”被更名为“setCompressionMode”/“getCompressionMode”
*   Rename filter被删除，取而代之的是更多特性的 “RenameUpload”。
*   StripTags filter：“setTagsAllowed”已更名为“setAllowedTags”，“setAttributesAllowed”已更名为“setAllowedAttributes”。
*   Encrypt/decrypt filter:不再​​压缩/解压缩。 你应该为添加了另一个filter实现。
*   Encrypt/decrypt filter 已经移到了Zend\Crypt命名空间。

### Validators(验证)

*   验证现在无状态的。
*   各种选项(options)现在使用下划线分离(underscore_separated)，（比如“messageTemplates”变成“message_templates”）。
*   验证程序不再有“的isValid”的方法。 相反，你必须在每个validator里调用“validate”的方法，返回一个ValidationResult对象。 这是可序列化(serializable)，可翻译(translatable)，并提供“的isValid”的方法。
*   Validators不再提供一个选项来设置消息的最小长度。 这应该由视图助手处理。
*   DateStep Validator不再继承 Date validator。
*   CreditCard Validator：option "type" 已更名为“allowed_types”，以更好地反映它做什么

### Event manager(事件管理器)

*   propagationIsStopped方法已重命名为isPropagationStopped。
*   StaticEventManager已被删除。
*   You cannot any longer emulate triggerUntil with trigger method. Use triggerUntil only if you need a callback.
*   “attach”的方法不再接受ListenerAggregateInterface。 使用attachAggregate方法来代替。

### RBAC

*   Rbac 和 Role 不在实现 RecursiveIterator接口，相反，RoleInterface实现更简单和更高效的IteratorAggregate接口。但是，这只是内部变化，不会影响任何人操作，作为一个副作用，AbstractIterator类不复存在。
*   AbstractRole的“addRole”的方法不再接受一个字符串：你必须给一个RoleInterface（ZF2里如果创建的角色是一个字符串，会自动创建Role对象。 但是，仍然支持直接在RBAC容器用字符串添加的角色。 所以这个变化是非常小的。

### Hydrators

### Forms

### Service manager

### Module manager
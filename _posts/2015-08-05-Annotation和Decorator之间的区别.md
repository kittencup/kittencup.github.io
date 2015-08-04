---
layout: post
title: "Annotation和Decorator之间的区别"
date: 2015-08-05
categories: javascript
---

去年，Angular团队宣布了ECMAScript的语言扩展AtScript，它增加了类型(Type)和注解(Annotation)功能，以便能够更好调试和开发，半年后，在ng-conf上，
团队宣布之前的AtScript变为了TypeScript，支持Annotation及另一个特性Decorator。

但是这些Annotation是如何工作的呢？什么是Decorator呢？本文讲解了Annotation和Decorator之间的区别

#### Annotations

让我们从Annotation开始。如前所述，Angular团队宣布他们使用AtScript来扩展JavaScript语言。
AtScript带有Type Annotation，Field Annotation和MetaData Annotation功能。我们将集中讨论Metadata Annotation。让我们来看看下面Angular 2组件的Metadata Annotation：

```javascript
@Component({
  selector: 'tabs'
})
@View({
  template: `
    <ul>
      <li>Tab 1</li>
      <li>Tab 2</li>
    </ul>
  `
})
export class Tabs {
}
```

我们有一个空的Tabs类，这个类有两个Annotation，@Component 和 @View,如果我们删除了所有的Annotation，剩下的只是一个没有任何特殊意义的空类?如此看来,@Component和@View为这个空的类添加一些元数据，以给它一个特定的含义。这就是Annotation,他们是以一个声明的方式将元数据添加到代码中。

@Component这个Annotation告诉Angular,这个类是一个组件，这个@View，给出这个组件关于视图相关信息，在这里，他是一个HTML模板。

好吧，这似乎看起来很清楚，有几个问题要提出：

* 这些Annotation来自哪里？JavaScript没有给我们这功能对吗?
* 谁定义了这些@Component和@View的Annotation？
* 如果这是AtScript的一部分,我们怎么将他编译成能在当今的浏览器运行的代码?

让我们逐一回答。这些Annotation来自哪里？要回答这个问题，我们需要完成示例代码。@Component和@View是我们需要从Angular 2框架中导入的东西：

```javascript
import { Component, View } from 'angular2/angular2';
```

这几乎回答了我们的第一个问题。Annotation是框架提供的。让我们看看这些Annotation的实现是什么样子:

```javascript
export class Component extends Directive {
  constructor() {
    ...
  }
}
```

我们可以看到，@Component 和 @View 实际上是Angular 框架来实现的。这回答了我们第二个问题。

等等。这只是另一个类？一个简单的类如何改变其他类的行为方式？为什么我们用@符号使这些类作为Annotation，并能够使用它们?恩，其实在当今浏览器中我们不能直接使用这些Annotation，这意味着我们需要转换(transpilers)它们成为能运行在当今浏览器上的代码。

尽管有几个transpilers我们可以从中选择,Babel, Traceur, TypeScript,事实证明只有Traceur是真正实现了从AtScript转换Annotation。上面的组件代码，使用Traceur将它转化为：

```javascript
var Tabs = (function () {
  function Tabs() {}
  Tabs.annotations = [
    new Component({...}),
    new View({...})
  ];
  return Tabs;
})
```

转换到最后，一个类只是一个函数，它也是一个对象，所有的Annotation最后都会成为这个对象的一个annotations属性里的实例，我们也可以有Parameter Annotation,最后都会变成这个函数parameters属性里的实例，
所以，如果我们有这样的代码：

```javascript
class MyClass {
  constructor(@Annotation() foo) {
    ...
  }
}
```

它将被转换为

```javascript
var MyClass = (function () {
  function MyClass() {}
  MyClass.parameters = [[new Annotation()]];
  return MyClass;
})
```

转化为一个嵌套数组的原因，是因为一个参数可以有一个以上的Annotation。

好吧，现在我们知道了这些Annotation是什么以及怎么来转换他们为浏览器能运行的代码，但我们仍然不知道像@Component如何让一个普通的类变为一个Angular 2组件。事实证明，Angular本身会处理这些。
Annotation真的只是将元数据添加到代码中。这就是为什么@Component和@View是由Angular 2来具体实现的。事实上，框架还有自带了其他几个Annotation，也只有框架本身知道如何处理这些信息。

另一个需要知道的是，Angular期望元数据在类的annotation和parameters属性上。如果Traceur不转换annotation到这些特殊的属性上时，Angular 2将不知道从哪里得到这些元数据。这使得AtScript Annotation只是一个非常具体的实际注释。

如果你作为一个消费者可以决定你的元数据附加到代码中的哪里? 是不是会更好，这就要使用Decorator

Decorator是由Yehuda Katz提出的 ECMAScript 7中建议的标准，让你可以在设计时对类和类的属性进行注解和修改，这听起来很像annotation做的事。好吧，让我们来看看什么是decorator

```javascript
// A simple decorator
@annotation
class MyClass { }
```

等等。这看起来酷似一个AtScript Annotation！ 对。但事实并非如此。从消费者的角度来看，一个Decorator确实看起来像我们所知道的“AtScript Annotation”。
但有一个显著差异。你需要负责装饰你的代码。上面的代码，相应的@annotation Decorator实现看起来应该是这样的：

```javascript
function annotation(target) {
   // Add a property on target
   target.annotated = true;
}
```

没错。decorator只是一个函数，让你访问一个需要被装饰的目标。明白了吧？而不是由transpiler来决定你的注释应该怎么转换，我们是负责定义具体的decorator

当前，我们也可以实现一个decorator同AtScript Annotations一样为我们的代码添加元数据(我一直提到“AtScript Annotation”,因为他们所做的事情,确实是一个AtScript具体的事情)，
或者换句话说，通过decorator我们可以创建Annotations

还有很多关于decorators的探讨，但这超出了本文的范围，你可以通过查看(Yehuda’s)[https://github.com/wycats/javascript-decorators]提出的建议来学习更多特性

#### TypeScript是否支持Annotation或Decorator？

正如你可能知道，Angular团队今年早些时候宣布，他们为了支持TypeScript准备放弃AtScript,因为这两种语言似乎解决了同样的问题。此外，有公告，版本1.5 alpha的TypeScript将支持Annotation和Decorator。

TypeScript支持Decorator,但是不知道关于Angular 2具体的Annotation,这是有道理的,因为这些Annotation是Angular来实现细节的，这也意味着，无论是作为消费者的我们，或框架为了使代码编译需要提供这些Decorator。
幸运的是，annotation和parameter,decorators在最近的Angular 2的代码库都已经实现了

#### 结论

AtScript Annotation 和 decorator 几乎是一样的.我们从消费者的角度来看有完全相同的语法。唯一的区别就是，我们没有去控制AtScript Annotation如何将元数据添加到我们的代码中，而decorator是对这些annotation最终的实现。
然而，从长远来看，我们可以只关注decorator,因为这是真正的标准建议，AtScript是TypeScript,TypeScript来实现Decorator

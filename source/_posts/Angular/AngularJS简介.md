title: 一.简介
date: 2016-07-31 15:45:21
tags: [Angular]
---

### 1.AugularJS简介
---
AngularJS 是一个为动态WEB应用设计的结构框架，提供给大家一种新的开发应用方式，这种方式可以让你扩展HTML的语法，以弥补在构建动态WEB应用时静态文本的不足，从而在web应用程序中使用HTML声明动态内容。

AngularJS有五个主要核心特性，如下介绍：
* 双向数据绑定 —— 实现了把model与view完全绑定在一起，model变化，view也变化，反之亦然。

* 模板 —— 在AngularJS中，模板相当于HTML文件被浏览器解析到DOM中，AngularJS遍历这些DOM，也就是说AuguarJS把模板当做DOM来操作，去生成一些指令来完成对view的数据绑定。

* MVVM —— 吸收了传统的MVC设计模式针但又并不执行传统意义上的MVC，更接近于MVVM(Moodel-View-ViewModel)。

* 依赖注入 —— AngularJS拥有内建的依赖注入子系统，可以帮助开发人员更容易的开发，理解和测试应用。

* 指令 —— 可以用来创建自定义的标签，也可以用来装饰元素或者操作DOM属性

### 2.引入AngularJS脚本
---
认识了AngularJS框架，我们开始创建第一个AngularJS应用。

AngularJS是以一个JavaScript文件形式发布的，可通过script标签载入AngularJS脚本，如下所示：

``` 
<script src="http://code.angularjs.org/angular-1.0.1.min.js"></script> 
```

完整的html代码如下：
```
<!doctype html>
<html ng-app>
    <head>
        <script src="http://code.angularjs.org/angular-1.0.1.min.js"></script>
    </head>
    <body>
        Hello {{'World'}}!
    </body>
</html>
```
请在您的浏览器中运行以上代码查看效果。 

### 3.AngularJS指令
---
AngularJS有一套完整的、可扩展的、用来帮助web应用开发的指令集，它使得HTML可以转变成“特定领域语言(DSL)”，是用来扩展浏览器能力的技术之一，在DOM编译期间，和HTML关联着的指令会被检测到，并且被执行，这使得指令可以为DOM指定行为，或者改变它。

AngularJS通过称为指令的新属性来扩展的HTML，带有前缀ng-，我们也可以称之为“指令属性”，它就是绑定在DOM元素上的函数，可以调用方法、定义行为、绑定controller及$scope对象、操作DOM，等等。

AngularJS指令指示的是“当关联的HTML结构进入编译阶段时应该执行的操作”，它本质上只是一个当编译器编译到相关DOM时需要执行的函数，可以写在元素的名称里，属性里，css类名里，注释里。

当浏览器启动、开始解析HTML时，DOM元素上的指令属性就会跟其他属性一样被解析，也就是说当一个Angular.js应用启动，Angular编译器就会遍历DOM树来解析HTML，寻找这些指令属性函数，在一个DOM元素上找到一个或多个这样的指令属性函数，它们就会被收集起来、排序，然后按照优先级顺序被执行。

Angular.js应用的动态性和响应能力，都要归功于指令属性，常见的有：ng-app、ng-init、ng-model、ng-bind、ng-repeat等等。

### 4.指令：ng-app
---
ng-app指令来标明一个AngularJS应用程序，并通过AngularJS完成自动初始化应用和标记应用根作用域，同时载入和指令内容相关的模块，并通过拥有ng-app指令的标签为根节点开始编译其中的DOM。

引用方法很简单，如下所示：

	<div ng-app="">
	</div>  


如上引用，一个AngularJS应用程序初始化就完成了并标记了作用域，也就是div元素就是AngularJS应用程序的所有者，在它里面的指令也就会被Angular编译器所编译、解析了。

### 5.指令：ng-init
---
g-init指令初始化应用程序数据，也就是为AngularJS应用程序定义初始值，通常情况下，我们会使用一个控制器或模块来代替它，后面我们会介绍有关控制器和模块的知识。

如下所示，我们为应用程序变量name赋定初始值。

	<div ng-app="" ng-init="name='Hello World'">
	</div>


我们不仅可以赋值字符串，也可以赋值为数字、数组、对象，而且可以为多个变量赋初始值，如下所示：

	<div ng-app="" ng-init="quantity=1;price=5">
	</div>


//或者

	<div ng-app="" ng-init="names=['Tom','Jerry','Gaffey']">
	</div>


### 6.数据绑定：表达式
---
AngularJS框架的核心功能之一 —— 数据绑定，由两个花括号组成，可以把数据绑定到HTML，类似Javascript代码片段，可以包含文字、运算符和变量，通常在绑定数据中用到，表达式可以绑定数字、字符串、对象、数组，写在双大括号内。

如前面的示例，我们就可以使用表达式这样调用初始化的变量值，如下。
```
	<div ng-app="" ng-init="name='Hello World'">
  		{{ name }}
	</div>
```

当然我们也可以使用表达式输出数字、数组等等，如下所示：
2.1. 输出数字，如下示例：

	<div ng-app="" ng-init="quantity=12;price=5">
		总价： {{ quantity * price }}
	</div>


2.2. 输出对象，如下示例：

	<div ng-app="" ng-init="names=['Tom','Jerry','Gaffey']">
		名字为： {{ names[0] }}
	</div>


### 7.指令：ng-model
---
在AngularJS中，只需要使用ng-model指令就可以把应用程序数据绑定到HTML元素，实现model和view的双向绑定。

如下示例，使用ng-model指令对数据进行绑定。

	<div ng-app="">
		请输入任意值：<input type="text" ng-model="name" />
		你输入的为： {{ name }}
	</div>


ng-model把相关处理事件绑定到指定标签上，这样我们就可以不用在手工处理相关事件(比如change等)的条件下完成对数据的展现需求。

### 8.数据绑定：ng-bind
---
指令ng-bind和AngularJS表达式有异曲同工之妙，但不同之处就在于ng-bind是在angular解析渲染完毕后才将数据显示出来的。

如下使用ng-bind指令绑定把应用程序数据。

	<div ng-app="">
			请输入一个名字：<input type="text" ng-model="name" />
			Hello <span ng-bind="name"></span>
	</div>


PS：使用花括号语法时，因为浏览器需要首先加载页面，渲染它，然后AngularJS才能把它解析成你期望看到的内容，所以对于首个页面中的数据绑定操作，建议采用ng-bind，以避免其未被渲染的模板被用户看到。

### 9.指令：ng-click
---
AngularJS也有自己的HTML事件指令,比如说通过ng-click定义一个AngularJS单击事件。

对按钮、链接等，我们都可以用ng-click指令属性来实现绑定，如下简单示例：
```
<div ng-app="" ng-init="click=false">
    <button ng-click="click= !click">隐藏/显示</button>
    <div ng-hide="click">
        请输入一个名字：<input type="text" ng-model="name" />
        Hello <span ng-bind="name"></span> 
    </div>
</div>
```

PS：ng-hide="true"，设置HTML元素不可见。

ng-click指令将DOM元素的鼠标点击事件(即mousedown)绑定到一个方法上，当浏览器在该DOM元素上鼠标触发点击事件时，Angular就会调用相应的方法，是不是很简单方便呢！
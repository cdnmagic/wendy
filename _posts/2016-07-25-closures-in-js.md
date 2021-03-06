---
layout: post
title: "JavaScript中闭包详解"
keywords: ["JavaScript","闭包","js","Closures"]
description: "详细解读JavaScript中的闭包"
category: "JavaScript"
tags: ["JavaScript","Closures"]
---
{% include JB/setup %}

##闭包

计算机科学中，闭包（Closure）又称词法闭包（Lexical Closure）或函数闭包，是引用了自由变量的函数。这个被引用的自由变量和将函数一同存在，即使离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组成的实体。闭包运行时可有多个实例，不同的引用环境和相同的函数组合可产生不同的实例。

在一些语言中，在函数可以定义（嵌套）另一个函数时，若内部函数引用了来自外部函数的变量，则可能产生闭包。运行时，一旦外部函数被执行，一个闭包就形成了。闭包包含了内部函数的代码，以及所需外部函数变量的引用。

<i style="float:right">——引用自“维基百科”</i><br/>

来解释一下与闭包定义相关的几个概念吧。

###自由变量

<span class="txt">在作用域 A 中使用的变量 x，没有在 A 而是在其他作用域中声明，那么对作用域 A 来说，x 就是一个自由变量。</span>

<pre>
function b(){
	var x = "Selina";
	function a(y){
		alert(x);	//对函数a()来说，x就是自由变量
	}
}
</pre>

观察上面的例子，可以发现，对作用域为 A 的一个函数，它的自由变量 x 既不是参数也不是该函数的局部变量。

再来看另外一段代码：

<pre>
var x = "Selina,out of bar()";
function foo(){
	alert(x);
}
function bar(fn){
	var x = "Hebe,inside bar()";
	(function(){
		x = "Ella";
		fn();
	})();
}
bar(foo());
</pre>

调用`bar(foo())`后会弹出什么结果呢？答案是`"Selina,out of bar()"`。也就是说，对于函数`foo()`而言，不管它在哪里被调用，取其中自由变量的值的时候，都要到创建`foo()`的作用域中取值，而不是调用它的作用域。造成这样的结果的原因其实在作用域链那篇文章中也提到过，因为创建一个函数的时候，它的作用域就已经确定了，而不是调用它的时候。

###作用域链

函数执行环境的作用域链在函数调用时创建，包含这个函数的活动对象和函数的[[scope]]属性。

`[[scope]]`是所有父级变量对象的层级链。在函数创建时即作为函数的静态属性存在，不可更改，一直存在直至函数被销毁（即使函数不被调用，它的`[[scope]]`属性也是存在的）。
PS.通过构造函数创建的函数是例外，它们的`[[scope]]`属性总是唯一的全局对象。

下面是函数被调用时作用域链的构成：

<pre>作用域链 = 执行环境活动对象|变量对象 + [[scope]];</pre>

如果你对这个知识点还不太了解，可移步 [JavaScript中的执行环境与作用域链](http://blog.hardworking.top/javascript/scope-in-js.html)。

###JavaScript中的闭包

理论上来说，JavaScript 中的所有函数都是闭包，因为它们在创建时都保存了上层函数的作用域链（即`[[scope]]`）。

然而，从实践上来说，满足条件 *①即使创建环境被销毁，依然存在 ②代码中引用了自由变量* 的函数才是闭包。

<pre>
var x = "Selina";
function foo(){
	alert(x);
}

//foo()是闭包
foo: <FunctionObject> = {
	[[scope]]: [
		global: {
			x: "Selina"
		}
	],
	...
}
</pre>

需要注意的是，同一个执行环境中，`[[scope]]`属性是被所有闭包共享的。也就是说，当其中一个闭包对`[[scope]]`的某变量作出修改，会影响到其它闭包对该变量的读取。如下例：

<pre>
var firstClosure,secondClosure;
function foo(){
    var x = 10;
    firstClosure = function (){
        return "firstClosure: " +(++x);
    }
    secondClosure = function (){
        return "secondClosure: " +(--x);
    }
    x = 20;
    alert(firstClosure());	//"firstClosure: 21"
}
foo();
alert(firstClosure());	//"firstClosure: 22"
alert(secondClosure());	//"secondClosure: 21"
</pre>

因为闭包保存的是整个变量对象，而不是某个特殊的变量，由于作用域链的配置机制，它会带来一个副作用，即闭包只能取得包含函数中任何变量的最后一个值。如下例：

<pre>
function foo(k,max){
    var result = new Array();
    for (var i = 0; i < max; i++){
        result[i] = function(){
            alert(i);
        };
    }
    return result[k]();
}

foo(1,5);	//5
foo(2,5);	//5
foo(4,5);	//5
</pre>

表面上看，函数似乎会返回自己的索引值即 k 值（循环中为i），其实则不然，每个函数都返回了传入的 max 值。因为每个函数的作用域链中都保存着`foo()`函数的活动对象，所以它们引用的都是同一个变量 i。当`foo()`返回后，i 的值为 max。

我们可以创建一个匿名函数强制让闭包的行为符合预期：

<pre>
function foo(k,max){
    var result = new Array();
    for (var i = 0; i < max; i++){
        result[i] = function(num){
            return function(){
                alert(num);
            };
        }(i);
    }
    return result[k]();
}

foo(1,5);	//1
foo(2,5);	//2
foo(4,5);	//4
</pre>

这样重写后，我们没有把闭包直接赋值给数组，而是定义了一个匿名函数，将立即执行这个匿名函数的结果赋给数组。在调用每个匿名函数时，传入变量 i，因为函数参数是按值传递的，所以会把 i 的当前值复制给 num。

JavaScript 中，闭包的返回语句会将控制流返回给调用者。

由于闭包会携带包含它的函数作用域，因此会比其它函数占用更多内存。在实际生产环境中，应避免闭包的滥用。

###关于 this 对象

闭包中使用 this 对象可能发生一些问题。我们知道，this 是函数运行时基于函数的执行环境绑定的：全局函数中，this === window；当函数被作为某个对象的方法调用时， this 等于那个对象。不过，匿名函数的执行具有全局性，因此 this 通常指向 window。

观察下面两个 🌰：

<pre>
name = "this is window";
var obj = {
    name: "this is obj",
    getName: function(){
        return function(){
            return this.name;
        }
    }
};
alert(obj.getName()());	//"this is window"
</pre>

<pre>
name = "this is window";
var obj = {
    name: "this is obj",
    getName: function(){
        var _this = this;
        return function(){
            return _this.name;
        }
    }
};
alert(obj.getName()());	//"this is obj"
</pre>

第一段代码的`getName()`返回一个匿名函数，其 this 指向 window。为什么匿名函数没有取得其外部作用域的 this 对象呢？

每个函数在被调用时会自动取得两个特殊对象：this 和 arguments。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，因此永远不会访问到外部函数的这两个变量。

##测试

看本文中代码的演示效果，请移步[这里](http://blog.hardworking.top/example/closures/)。

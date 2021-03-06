---
layout: post
title: "Symmetric Difference(算法)"
keywords: ["FCC ","算法","freecodecamp","Symmetric Difference"]
description: "FCC算法题 Symmetric Difference"
category: "FCC"
tags: ["FCC","中级算法","algorithm","Array"]
---
{% include JB/setup %}

###题目

创建一个函数，接受两个或多个数组，返回所给数组的<span class="txt"> 对等差分(symmetric difference) </span>(△ or ⊕)数组.

给出两个集合 (如集合 A = {1, 2, 3} 和集合 B = {2, 3, 4}), 而数学术语 "对等差分" 的集合就是指由所有只在两个集合其中之一的元素组成的集合(A △ B = C = {1, 4}). 对于传入的额外集合 (如 D = {2, 3}), 你应该安装前面原则求前两个集合的结果与新集合的对等差分集合 (C △ D = {1, 4} △ {2, 3} = {1, 2, 3, 4}).

###提示

[Array.reduce()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

[Symmetric Difference](https://www.youtube.com/watch?v=PxffSUQRkG4)

###思路

问：怎么把大象装进冰箱里？第一步，把冰箱门打开。那么怎么解决这道题目？第一步，拿到所有可能传入的作为参数的待处理数组。

一看到那些未显示声明或者不确定个数的参数，就能想到要好好利用一下`arguments`这个ECMAScript中的特殊对象了。

>在函数代码中，使用特殊对象 arguments，开发者无需明确指出参数名，就能访问它们。

接下来，把大象装进冰箱里，哦，不，是把`arguments`对象的集合放进一个新数组。

有两种方法可以办到。第一种，老生常谈的`for()`循环。

<pre>
var arr = [];
  for(var i = 0; i < arguments.length; i++){
    arr.push(arguments[i]);
  }
</pre>

第二种是使用 ECMA-262 第六版标准添加的`Array.from()`方法。它可以将两类对象转为真正的数组：<span class="txt">类似数组的对象（array-like object）</span>和<span class="txt">可遍历（iterable）的对象</span>。

<pre>
var arr = Array.from(arguments);
</pre>

通读题目，其实就是要求我们把参数数组中的前两项做运算，得到一个所有项都没同时出现在两个数组中的新数组，然后把这个新数组作为首项，继续与后面的数组进行同样操作。

我们第一步得到的参数数组是个二维数组，而函数最后返回的是一个结果，一维数组，很容易就想到Array归并用的`reduce()`方法。

`reduce()`传入的函数带有4个参数：前一个值`prev`、当前值`cur`、项的索引`index`、数组对象`array`。

分别对`prev`和`cur`进行过滤，返回对方数组中不存在的值，连接成为新数组。最后对新数组去重。

去重时，可以另写一个函数，像这样：

<pre>
function unique(array){
    var n = [];
    for(var i = 0;i < array.length; i++){
        if(n.indexOf(array[i]) == -1) {
       	 n.push(array[i]);
        }
    }
    return n;
}
</pre>

也可以在返回结果中继续利用`filter()`去重：

<pre>
//整个函数的返回值
return temp.filter(function(item,index,array){
    return array.indexOf(item) == index;
  });
</pre>

###解法

<pre>
function sym(args) {
  var arr = [];
  for(var i = 0; i < arguments.length; i++){
    arr.push(arguments[i]);
  }
  var temp = arr.reduce(function(prev,cur,index,array){
    var a = prev.filter(function(item){
      return cur.indexOf(item) < 0;
    });
    var b = cur.filter(function(item){
      return prev.indexOf(item) < 0;
    });
    return a.concat(b);
  });
  return temp.filter(function(item,index,array){
    return array.indexOf(item) == index;
  });
  //或者调用外部函数去重；function unique(array)见“思路”部分
  //return unique(temp);
}
</pre>

###测试

`sym([1, 2, 3], [5, 2, 1, 4])` 应该返回 [3, 4, 5].

`sym([1, 2, 3], [5, 2, 1, 4])` 应该只包含三个元素.

`sym([1, 2, 5], [2, 3, 5], [3, 4, 5])` 应该返回 [1, 4, 5]

`sym([1, 2, 5], [2, 3, 5], [3, 4, 5])` 应该只包含三个元素.

`sym([1, 1, 2, 5], [2, 2, 3, 5], [3, 4, 5, 5])` 应该返回 [1, 4, 5].

`sym([1, 1, 2, 5], [2, 2, 3, 5], [3, 4, 5, 5])` 应该只包含三个元素.

`sym([3, 3, 3, 2, 5], [2, 1, 5, 7], [3, 4, 6, 6], [1, 2, 3])` 应该返回 [2, 3, 4, 6, 7].

`sym([3, 3, 3, 2, 5], [2, 1, 5, 7], [3, 4, 6, 6], [1, 2, 3])` 应该只包含五个元素.

`sym([3, 3, 3, 2, 5], [2, 1, 5, 7], [3, 4, 6, 6], [1, 2, 3], [5, 3, 9, 8], [1])` 应该返回 [1, 2, 4, 5, 6, 7, 8, 9].

`sym([3, 3, 3, 2, 5], [2, 1, 5, 7], [3, 4, 6, 6], [1, 2, 3], [5, 3, 9, 8], [1])` 应该只包含八个元素.
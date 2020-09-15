---
layout: post
title: "[译]简明易懂javascript闭包"
date: 2018-05-27 18:22:04 +0800
categories: javascript基础
banner_img: /images/javascript-closure-banner.jpg
index_img: /images/javascript-closure-banner.jpg
tags: javascript
---

> 今天带来一篇关于 javacript 闭包的文章，文章来自 stackoverflow 的一个高票回答，个人感觉非常不错，兴致来了翻译一下，方便大家和我日后回顾，也顺便锻炼下我的英文水平:)。原文地址[在这](https://stackoverflow.com/questions/111102/how-do-javascript-closures-work)，本文目的只是学习研究，如有不妥当之处，请通过邮件告知我，非常感谢！

### 那么，正文开始～

#### 一、闭包不是魔法

本文的目的是通过小段 js 代码为广大程序员说明闭包，目标群体并不是此方面的专家。
如果理解了闭包的核心概念，闭包并不是一个很难理解的问题。但是如果仅仅靠阅读学术文章是不可能理解闭包的。这篇文章需要一些基本的编程知识，如果你可以读懂下面这段代码，那么恭喜你，理解这篇文章对你来说不是问题。

```javascript
function sayHello(name) {
  var text = "Hello " + name;
  var say = function () {
    console.log(text);
  };
  say();
}
sayHello("Joe");
```

#### 二、一个栗子

闭包可以用两种方式来总结，即:

- 闭包是一种支持[first-class function](https://en.wikipedia.org/wiki/First-class_function)的方式。他可以引用其第一次声明时的范围内的变量，我们可以任意使用变量，赋值、当作参数传递，或者通过 return 返回都是可以的。
- 闭包是当一个函数开始执行的时候，js 内部自动分配的一种堆栈结构，即使函数返回了这个堆栈结构都不会被释放(这种数据结构是分配在堆上而不是栈上的!)。

如下代码返回了一个函数引用:

```javascript
function sayHello2(name) {
  var text = "Hello " + name; // Local variable
  var say = function () {
    console.log(text);
  };
  return say;
}
var say2 = sayHello2("Bob");
say2(); // logs "Hello Bob"
```

大多数 js 工程师都能理解上述代码中函数引用作为返回值的用法。如果你不能理解，那么你应该先了解一下这部分内容然后再继续学习闭包。一个 C 语言工程师可能会认为这个函数返回了另一个函数的指针，变量`say`和`say2`都是函数指针。
但是在 js 中上述代码和函数指针有一个重大的不同。在 js 中，你可以把函数引用理解为**函数指针+闭包的隐形引用**。
上述代码由于匿名函数`function() { console.log(text); }`声明在另一个函数(sayHello2)中，所以可以称之为闭包。在 javascript 中，如果你在一个方法中使用`function`这个关键字，你就创建了一个闭包。
在大部分语言中，在一个函数返回后，所有局部变量就都不能访问了，因为他们的堆栈结构都被销毁了。
在 js 中，如果你在一个函数中定义另一个函数，在外部函数返回后，外部函数的局部变量仍可以被内部函数访问，上述代码就是这个现象的演示。我们在`sayHello2()`函数返回后去调用`say2()`，注意`text`这个变量是`sayHello2()`的局部变量。

```javascript
function() { console.log(text); } // Output of say2.toString();
```

上述代码为 say2.toString()的值，我们可以看到上述代码中引用了变量`text`。这个匿名函数可以引用`text`这个变量(值为`Hello Bob`)，这是因为`sayHello()`的局部变量被保存在了闭包中。
神奇之处在于 js 中函数的引用会包含一个隐形的引用指向外部函数的闭包--相当于包含一个函数的指针+一个隐形的对象引用。

#### 三、更多的栗子

在你阅读闭包释义的时候很难理解其中奥妙，但是当你看一些例子之后就会明白他们是如何工作的。我建议把下列例子仔细弄明白。如果你在使用闭包之前没有对其有充分的理解，你可能会制造一些十分诡异的 bug。

##### Example 3

这个例子说明了局部变量并不是被拷贝的--闭包中存的是他们的引用。可以理解为当外部函数退出时，变量的堆栈结构被存储在了内存中。

```javascript
function say667() {
  // Local variable that ends up within closure
  var num = 42;
  var say = function () {
    console.log(num);
  };
  num++;
  return say;
}
var sayNumber = say667();
sayNumber(); // logs 43
```

##### Example 4

这三个全局函数持有同一个闭包的引用，因为他们都被定义在了函数`setupSomeGlobals()`中。

```javascript
var gLogNumber, gIncreaseNumber, gSetNumber;
function setupSomeGlobals() {
  // Local variable that ends up within closure
  var num = 42;
  // Store some references to functions as global variables
  gLogNumber = function () {
    console.log(num);
  };
  gIncreaseNumber = function () {
    num++;
  };
  gSetNumber = function (x) {
    num = x;
  };
}

setupSomeGlobals();
gIncreaseNumber();
gLogNumber(); // 43
gSetNumber(5);
gLogNumber(); // 5

var oldLog = gLogNumber;

setupSomeGlobals();
gLogNumber(); // 42

oldLog(); // 5
```

这三个函数共享同一个闭包--函数`setupSomeGlobals()`的局部变量。
注意在以上例子中，如果你又调用了一次`setupSomeGlobals()`函数，一个新的闭包(堆栈结构)就会被创建。旧的`gLogNumber`、`gIncreaseNumber`、`gSetNumber`这三个变量会被拥有新闭包的新函数覆盖。(在 js 中，当你创建这种嵌套 function 的关系后，每次调用外部函数都会生成新的闭包)。

##### Example 5

这个例子向我们说明了，闭包会涵盖外部函数结束前声明的所有局部变量。注意变量`alice`实际上是在匿名函数之后声明的。匿名函数先声明，但是在匿名函数之中可以访问`alice`变量(js 有[变量提升](https://stackoverflow.com/a/3725763/1269037)的机制)。`sayAlice()()`直接调用`sayAlice()`返回的函数引用-这样调用哦不需要临时变量。

> `say`这个变量也被保存在闭包中，所有在`sayAlice()`中声明的内部函数都可以调用他，甚至`say()`函数中也可以进行递归调用。

```javascript
function sayAlice() {
  var say = function () {
    console.log(alice);
  };
  // Local variable that ends up within closure
  var alice = "Hello Alice";
  return say;
}
sayAlice()(); // logs "Hello Alice"
```

##### Example 6

这个例子很多人已经理解原因了，所以你也要了解一下。当我们在循环中定义方法的时候要格外小心，闭包中包含的局部变量可能会出人意料。
在看这个例子之前，你要先理解之前提到的变量提升。

```javascript
function buildList(list) {
  var result = [];
  for (var i = 0; i < list.length; i++) {
    var item = "item" + i;
    result.push(function () {
      console.log(item + " " + list[i]);
    });
  }
  return result;
}

function testList() {
  var fnlist = buildList([1, 2, 3]);
  // Using j only to help prevent confusion -- could use i.
  for (var j = 0; j < fnlist.length; j++) {
    fnlist[j]();
  }
}

testList(); //logs "item2 undefined" 3 times
```

`result.push( function() {console.log(item + ' ' + list[i])} );`这行三次将匿名函数的引用加入到`result`数组中，如果你对匿名函数不了解，你可以将其看作如下代码：

```javascript
pointer = function () {
  console.log(item + " " + list[i]);
};
result.push(pointer);
```

这段代码的输出是三次`item2 undefined`!其中的原因类似之前的例子，这三个匿名函数包含同一个`buildList`闭包(其中包含`result`、`i`和`item`变量)。当`fnlist[j]();`这句代码运行的时候，他们使用的都是同一个闭包，同一组变量。这时`i`循环完毕，被累加到 3(`i++`),`item`变量一次次被覆盖，最后结果为`item2`。

当我们使用块级作用域声明`item`(通过`let`关键字)而不是使用函数作用域(通过`var`关键字)的时候，在`result`数组中存放的每个匿名函数都会有一个单独的闭包。输出会变为下面这样:

```javascript
item0 undefined
item1 undefined
item2 undefined
```

如果再把循环中使用的`i`通过`let`改为块级作用域的时候，我们会发现结果正常了:

```javascript
item0 1
item1 2
item2 3
```

##### Example 7

最后一个例子，每次调用`newClosure()`函数的时候都会新建一个单独的闭包。

```javascript
function newClosure(someNum, someRef) {
  // Local variables that end up within closure
  var num = someNum;
  var anArray = [1, 2, 3];
  var ref = someRef;
  return function (x) {
    num += x;
    anArray.push(num);
    console.log(
      "num: " +
        num +
        "; anArray: " +
        anArray.toString() +
        "; ref.someVar: " +
        ref.someVar +
        ";"
    );
  };
}
obj = { someVar: 4 };
fn1 = newClosure(4, obj);
fn2 = newClosure(5, obj);
fn1(1); // num: 5; anArray: 1,2,3,5; ref.someVar: 4;
fn2(1); // num: 6; anArray: 1,2,3,6; ref.someVar: 4;
obj.someVar++;
fn1(2); // num: 7; anArray: 1,2,3,5,7; ref.someVar: 5;
fn2(2); // num: 8; anArray: 1,2,3,6,8; ref.someVar: 5;
```

#### 四、总结

如果还有任何不清晰的地方，可以自己再研究运行下上面的例子。通过代码梳理思路比看释义要简单的多。我对闭包的堆栈结构之类的分析，在技术上来讲不完全准确--但这样解释可以明显的简化概念，便于理解。先搞懂大体的逻辑，具体的细节可以在日后使用的时候慢慢理解。

#### 五、知识点

- 每当你在一个函数中声明另一个函数中时，闭包就出现了。
- 当你在一个函数中使用`eval()`时，闭包出现。你在参数中可以调用外部函数的局部变量，在`eval`内你甚至可以通过`eval('var foo=...')`来创建新的局部变量。
- 当你在一个函数中使用`new Function(...)`(函数的[构造器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)，并不会产生闭包(在内部函数中不能使用外部函数的局部变量)。
- 闭包包含所有外部函数中的所有局部变量，只要在外部函数返回前声明的都会包含。
- 我们可以这么想，一个闭包只是创建了一个函数入口，局部变量被加入到闭包中。
- 每当外部函数被调用时，一个新的闭包被创建。
- 两个函数可能内容相同，但是运行出来的结果不同，这可能是因为隐藏的闭包。
- 函数的多层嵌套是可能的，你可能获得多级的闭包。(即内部函数的内部函数可以访问外部函数的局部变量)。
- 我通常认为闭包是函数声明和局部变量的集合体，虽然我在本篇文章中没有使用这个方式定义。
- 我怀疑 js 中的闭包和一般的函数式语言的实现方式不同。

#### 六、相关链接

- Douglas Crockford 通过闭包来实现[私有方法和私有变量的模拟](http://www.crockford.com/javascript/private.html)。
- 一个很好的[例子](https://www.codeproject.com/Articles/12231/Memory-Leakage-in-Internet-Explorer-revisited)，这个例子解释了闭包在 IE 中是如何导致内存泄漏的。

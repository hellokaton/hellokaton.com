---
layout: post
title: JS代码复用
categories: web
tags: javascript
---

**js 中复用代码**

说道代码复用，一般都会涉及到对象继承。在js中有许多可以选择的继承方法。这些方法对于学习和理解多种不同的模式有很大的好处，因为它们有助于提供对语言的掌握程度。

但是在开发的过程中，并不是所有的代码复用都会使用到继承。其中一部原因在于，事实上使用的js库可能以这样的或那样的方式解决了该问题。而另一方面的原因就在于很少需要在js中建立长而且复杂的继承链。在静态强类型语言中，继承可以能是唯一复用代码的方法。在js中，经常有更加简洁而且优美的方法。包括：借用方法、绑定、复制属性以及从多个对象中混入属性等许多方法。

<!-- more -->

**混入**

混入是针对通过属性复制实现继承的思想做进一步的扩展，mix-in模式并不是复制一个完整的对象，而是从多个对象中复制出任意的成员并将这些成员组合成新的对象。

实现mix-in：

```js
function mix() {
    var arg, prop, child = {};
    for (arg = 0; arg < arguments.length; arg += 1) {
        for (prop in arguments[arg]) {
            if (arguments[arg].hasOwnProperty(prop)) {
                child[prop] = arguemnts[arg][prop];
            }
        }
    }
    return child;
}
```
mix-in实现非常简单，只需要遍历每个参数，并且复制出传递给该函数的每个对象中的每个属性。

**借用方法**

有的时候，我们只需要对象中的一两个方法，但是又不想和对象形成父-子继承关系。只是想是用所需要的方法，而不希望继承所有那些永远用不到的属性和方法。在这种情况下，可以通过使用借用方法模式来实现。

而这个方法是受益于call()和apply()的。js中函数也是对象，并且它们自身也存在一些属性和方法，比如call和apply()。

使用call()和apply()分别借用方法：

```js
// call
notmyobj.doStuff.call(myobj, param1, p2, p3);

// apply
notmyobj.doStuff.apply(myobj, [param1, p2, p3]);
```
在知道notmyobj对象上有doStuff方法的情况下，又想不继承notmyobj来使用doStuff方法。这个使用call()和apply()就派上用场了。

还有一个场景是经常使用到借用方法的。那就是借用数组方法。因为数据具有许多很强大的方法，而且有时候需要操作arguments的时候会用上。但是arguments是一个伪数组，不具有原生数组强大的方法。这个使用借用方法就派上用场了：

```js
function f() {
    var args = [].slice.call(arguemnts, 1, 3);
    return args;
}
```

**借用和绑定**

考虑到借用方法不是通过调用call和apply()就是通过简单的复制，在借用方法的内部，this所指向的对象是基于调用表达式而确定的，但是有的时候“锁定”this的值，或者将其绑定到特定的对象并且预先确定该对象。

**举一栗子：**

```js
var one = {
    name: 'object',
    say: function (greet) {
        return greet + ", " + this.name;
    }
};
```

// 测试

```js
one.say('hi'); // hi, object
```

接着另一对象two中没有say()方法，借用one的say()方法：

```js
var two = {
    name: 'another object'
}
```

// 测试
one.say.call(two, 'hi'); // hi, another object
在上面的例子中，借用的say()方法的this是指向two的。但是在什么样的场景中，应该将函数指针赋值给一个全局变量，或者将该函数做为回调函数来传递？在客户端编程中有许多事件和回调，因此确实发生了许多这样混淆的事件。

举一个栗子：

```js
// 给变量赋值
// this 将指向全局变量
var say = one.say;
say('hello') // hello, undefined

// 作为回调传递
var yetanother = {
    name: 'yet another object',
    method: function (callback) {
        return callback('hola');
    }
};
```

// 测试

```js
yetanother.method(one.say) // holla, undefined
```

在上面的两种情况下，say()方法的this值都是指向全局对象。而且整个代码都无法按照预期来运行。为了修复（绑定）对象与方法之间的关系。可以使用一个简单的函数来实现：

```js
function bind(o, m) {
    return function () {
        return m.apply(o, [].slice.call(arguments))
    }
}
```
上面的bind()方法接受两个参数。一个是对象o，另一个是方法m，并将两者绑定起来。然后返回一个函数。

使用bind()来解决问题：

```js
var twosay = bind(two, one.say);
twosay('yo'); // yo another object
```

奢侈的拥有绑定所需要辅助的代价就是额外的闭包的开销。
ES5 bind()

在ECMAScript5中给Function.protoype添加了一个bind()方法，使得bind()与call()、apply()一样简单易用。

但是在不支持ECMAScript5的运行环境下，我们可以自己实现一个bind()方法（来自 MDN）：

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      // closest thing possible to the ECMAScript 5 internal IsCallable function
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(this instanceof fNOP && oThis
                                 ? this
                                 : oThis || window,
                               aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

然后使用自带的bind()方法来重写一下上面的栗子：

```js
var twosay = bind(two, one.say);
twosay('Bonjour'); // yo another object
```
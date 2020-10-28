在绝大多数的情况下，普通函数的this返回的值取决于调用环境，箭头函数的this在定义时已经决定。此外，在严格模式和非严格模式间也会有不同的表现。



#### 全局环境

无论是否在严格模式下，在**全局执行环境中（在任何函数体外部**）`this` 都指向全局对象。



#### 函数环境

在函数内部，`this`的值取决于被调用的方式。

##### 简单调用

非严格模式：并非调用设置，所以取默认值，为全局对象

严格模式：始终保持由调用环境设置，默认值为undefined

```js
function f1(){
    return this;
}
function f2(){
    "use strict";
    return this;
}

//在浏览器中：
f1() === window; //true
//在Node中：
f1() === global;  //true
f2() === undefined; //true
```

##### 将函数this绑定到特定对象

```js
function.call(thisArg, arg1, arg2, ...)
function.apply(thisArg, [argsArray])
```

使用 `call` 和 `apply` 函数的时候要注意，如果传递给 `thisArg` 的值不是一个对象，JavaScript 会尝试使用内部 `ToObject` 操作将其转换为对象。

##### bind

```js
function.bind(thisArg[, arg1[, arg2[, ...]]])
```

创建一个新的**绑定函数**（**bound function**，BF）。绑定函数是一个 exotic function object（怪异函数对象，ECMAScript 2015 中的术语），它包装了原函数对象。调用**绑定函数**通常会导致执行**包装函数**。

绑定函数无法再使用bind方法，如下

```js
function f(){
  return this.a;
}

var g = f.bind({a:"azerty"});
console.log(g()); // azerty

var t = f.bind({a:"test"});
console.log(t()); // test

var h = g.bind({a:'yoo'}); // bind只生效一次！
console.log(h()); // azerty

var o = {a:37, f:f, g:g, h:h,t:t};
console.log(o.a, o.f(), o.g(), o.h(),o.t()); // 37, 37, azerty, azerty, test
```



##### 箭头函数

在[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)中，`this`与封闭词法环境的`this`保持一致。在全局代码中，它将被设置为全局对象：

如果将`this`传递给`call`、`bind`、或者`apply`来调用箭头函数，它将被忽略。不过你仍然可以为调用添加参数，不过第一个参数（`thisArg`）应该设置为`null`。

```js
// 创建一个含有bar方法的obj对象，
// bar返回一个函数，
// 这个函数返回this，
// 这个返回的函数是以箭头函数创建的，
// 所以它的this被永久绑定到了它外层函数的this。
// bar的值可以在调用中设置，这反过来又设置了返回函数的值。
var obj = {
  bar: function() {
    var x = (() => this);
    return x;
  }
};

// 作为obj对象的一个方法来调用bar，把它的this绑定到obj。
// 将返回的函数的引用赋值给fn。
var fn = obj.bar();

// 直接调用fn而不设置this，
// 通常(即不使用箭头函数的情况)默认为全局对象
// 若在严格模式则为undefined
console.log(fn() === obj); // true

// 但是注意，如果你只是引用obj的方法，
// 而没有调用它
var fn2 = obj.bar;
// 那么调用箭头函数后，this指向window，因为它从 bar 继承了this。
console.log(fn2()() == window); // true
```

在上面的例子中，一个赋值给了 `obj.bar`的函数（称为匿名函数 A），返回了另一个箭头函数（称为匿名函数 B）。因此，在 `A` 调用时，函数B的`this`被永久设置为obj.bar（函数A）的`this`。当返回的函数（函数B）被调用时，它`this`始终是最初设置的。在上面的代码示例中，函数B的`this`被设置为函数A的`this`，即obj，所以即使被调用的方式通常将其设置为 `undefined` 或全局对象（或者如前面示例中的其他全局执行环境中的方法），它的 `this` 也仍然是 `obj` 。



##### 作为对象的方法

当函数作为对象里的方法被调用时，它们的 `this` 是调用该函数的对象。

```js
var o = {prop: 37};
var prop = 50;

function a() {
  return this.prop;
}
var b = (()=> this.prop);

o.a = a;
o.b = b;

console.log(o.a()); // 37
console.log(o.b()); // 55
```



##### 原型链中的this

对于在对象原型链上某处定义的方法，同样的概念也适用。如果该方法存在于一个对象的原型链上，那么`this`指向的是调用这个方法的对象，就像该方法在对象上一样。



##### getter 与 setter 中的 this

再次，相同的概念也适用于当函数在一个 `getter` 或者 `setter` 中被调用。用作 `getter` 或 `setter` 的函数都会把 `this` 绑定到设置或获取属性的对象。



##### 作为构造函数

```js
/*
 * 构造函数这样工作:
 *
 * function MyConstructor(){
 *   // 函数实体写在这里
 *   // 根据需要在this上创建属性，然后赋值给它们，比如：
 *   this.fum = "nom";
 *   // 等等...
 *
 *   // 如果函数具有返回对象的return语句，
 *   // 则该对象将是 new 表达式的结果。 
 *   // 否则，表达式的结果是当前绑定到 this 的对象。
 *   //（即通常看到的常见情况）。
 * }
 */

function C(){
  this.a = 37;
}

var o = new C();
console.log(o.a); // logs 37


function C2(){
  this.a = 37;
  this.b = “test”;
  return {a:38}; //指定返回一个对象，则只能从本对象里取值
}

o = new C2();
console.log(o.a); // logs 38
console.log(o.b); // undefined
```



##### 作为一个DOM事件处理函数

当函数被用作事件处理函数时，它的`this`指向触发事件的元素（一些浏览器在使用非`addEventListener`的函数动态添加监听函数时不遵守这个约定）。



##### 作为一个内联事件处理函数

当代码被内联[on-event 处理函数](https://developer.mozilla.org/zh-CN/docs/Web/Guide/Events/Event_handlers)调用时，它的`this`指向监听器所在的DOM元素：

```html
<button onclick="alert(this.tagName.toLowerCase());">
  Show this
</button>
```

上面的 alert 会显示`button`。注意只有外层代码中的`this`是这样设置的：

```html
<button onclick="alert((function(){return this})());">
  Show inner this
</button>
```

在这种情况下，没有设置内部函数的`this`，所以它指向 global/window 对象（即非严格模式下调用的函数未设置`this`时指向的默认对象）。





参考：[this - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)




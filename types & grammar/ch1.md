# 你不知道的JavaScript:类型和语法
# 第一章：类型

大多数开发人员认为，动态语言（如JavaScript）并没有*类型*。让我们来看看ES5.1的规范(http://www.ecma-international.org/ecma-262/5.1/)对于这部分内容是怎么说的：

> 本规范中所有算法所操作的值都有一个类型与之对应。这些值的类型均在本规范中对应。当然，这些类型也可能是ECMAScript语言中规定的类型的子类型。
>
> 在ECMAScript语言中，每个ECMAScript类型所对应的值都被ECMAScript程序开发人员直接操作。ECMAScript语言中规定的类型为Undefined， Null，Boolean，String，Number，以及Object。

如果你是强类型语言（静态语言）的粉丝，你也许会对这样使用“类型”感到很反感。在那些语言里，“类型”所拥有的含义可比在JS里的多得多。

有人说JS不应该声称它有“类型，应该把这种东西称为“标签”，或是“子类型”。

好吧。我们将使用这一粗略的定义（类似于规范中所描述的）：一个*类型*是一个固有的，内建的特征集，无论是编译引擎还是**开发人员**，都可以用它来确定一个值的行为，并把这个值和其他值加以区分。

简单来说，如果在编译引擎和开发人员眼里，值`42`（数字）和值`"42"`（字符串）处理的方法不同，那么我们就说他们有不同的*类型*——`number`和`string`。当你处理`42`时，你将使用一些处理数字的方法，比如数学运算。而当你处理`"42"`时，你则会使用一些字符串处理方法，比如输出到页面，等等。**这两个值有不同的类型。**

虽然这并不是什么严谨的定义，但对于我们接下来的讨论，已经绰绰有余了。而且这样的定义，和JS如何形容自己是一致的。

# 类型——或是别的什么

不考虑学术上的争论，我们来想想为什么JavaScript会需要*类型*？

对于每种*类型*及其基本行为都有所了解，有助于更高效的将值进行类型转换（详见第四章，类型转换）。几乎所有的JS程序，都存在着这样那样的类型转换，所以了解这些，对你来说很重要。

如果你有一个值为`42`的`number`，但想对它进行`string`类型的操作，如移除`1`位置的字符`"2"`，你最好先将这个值的类型从`number`转换为`string`。

这看似很简单。

但是进行这样的类型转换，有很多方式。有些方式很明确，很简单就能说出来龙去脉，并且也值得信赖.但如果你不够细心，类型转换可能以一种匪夷所思的方式展现在你面前。

类型转换可能是JavaScript最大的疑惑之一了。这点经常被视为这一语言的缺陷，是应该避免使用的。

由于有了对JavaScript类型的全面了解，我们希望能够说明为何类型转换的*坏名声*言过其实，甚至是不恰当的——我们会改变你的传统观点，让你看到类型转换的强大力量和实用性。不过首先，我们先来了解一下值和类型。

## 内建类型

JavaScript定义了七种内建类型：

* `null`
* `undefined`
* `boolean`
* `number`
* `string`
* `object`
* `symbol` —— ES6中新增

**提示：**以上类型，除`object`的被称为基本类型。

`typeof`运算符会检测所给值得类型，并返回以下其中字符串类型的值——然而奇怪的是，返回的结果和我们刚刚列出的的内建类型并不一一对应。

```js
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// ES6新增！
typeof Symbol()      === "symbol";    // true
```

列出的六种类型的值都会返回一个对应类型名称的字符串。`Symbol`是ES6中新增的数据类型，我们会在第三章详细介绍。

你也许注意到了，我将`null`从列表中除去了。因为他很特殊——当使用`typeof`运算符时，它表现的就像bug一样：

```js
typeof null === "object"; // true
```

如果它返回的是`"null"`的话，那可真是件好事，可惜的是，这个bug已经存在了20年，而且由于有太多的web程序依赖这一bug运行，修复这一bug的话，将会创造更多的bug，并且使很多web应用无法运行，所以估计将来也不会修复。

如果你想要确定一个`null`类型的值是这一类型，你需要使用复合判定：

```js
var a = null;

(!a && typeof a === "object"); // true
```

`null`是基本类型中唯一值表现的像false一样的类型（详见第四章），但如果运行`typeof`进行检查，返回的还是`"object"`。

那么，`typeof`返回的第七种字符串类型的值是什么？

```js
typeof function a(){ /* .. */ } === "function"; // true
```

单拍脑袋想的话，很容易理解`function`（函数）会是JS中顶级的内建类型，尤其是它针对`typeof`运算符的表现。然而，如果你阅读相关的标准，会发现它实际上是对象类型（`object`）的子类型。更确切的说，函数是一种“可以被调用的对象”——一类拥有名为`[[Call]]`的内建属性且可以被调用的对象。

函数实际上是对象这点其实很有用。最重要的一点就是，它可以有属性。例如：

```js
function a(b,c) {
	/* .. */
}
```

该函数具有一个`length`属性，值为函数形式参数的个数。

```js
a.length; // 2
```

本例中，函数声明中包括两个形参（`b`和`c`），所以“函数的长度”是`2`。

那么数组呢？他们也是JS内置的类型，会不会有什么特殊的表现？

```js
typeof [1,2,3] === "object"; // true
```

然而并没有，只是普通的对象罢了。一般将它们也视为对象的“子类型”（详见第三章），与普通对象不同的是，它们可以通过数字来序列化（就像普通对象那样可以通过字符串类型的key（键）来序列化一样），并且操作有可以自动更新的`length`属性。

## 值和对象

在JavaScript中，变量不具有类型——**值有类型**。变量可以在任何时刻保存任何值。

Another way to think about JS types is that JS doesn't have "type enforcement," in that the engine doesn't insist that a *variable* always holds values of the *same initial type* that it starts out with. A variable can, in one assignment statement, hold a `string`, and in the next hold a `number`, and so on.

The *value* `42` has an intrinsic type of `number`, and its *type* cannot be changed. Another value, like `"42"` with the `string` type, can be created *from* the `number` value `42` through a process called **coercion** (see Chapter 4).

If you use `typeof` against a variable, it's not asking "what's the type of the variable?" as it may seem, since JS variables have no types. Instead, it's asking "what's the type of the value *in* the variable?"

```js
var a = 42;
typeof a; // "number"

a = true;
typeof a; // "boolean"
```

The `typeof` operator always returns a string. So:

```js
typeof typeof 42; // "string"
```

The first `typeof 42` returns `"number"`, and `typeof "number"` is `"string"`.

### `undefined` vs "undeclared"

Variables that have no value *currently*, actually have the `undefined` value. Calling `typeof` against such variables will return `"undefined"`:

```js
var a;

typeof a; // "undefined"

var b = 42;
var c;

// later
b = c;

typeof b; // "undefined"
typeof c; // "undefined"
```

It's tempting for most developers to think of the word "undefined" and think of it as a synonym for "undeclared." However, in JS, these two concepts are quite different.

An "undefined" variable is one that has been declared in the accessible scope, but *at the moment* has no other value in it. By contrast, an "undeclared" variable is one that has not been formally declared in the accessible scope.

Consider:

```js
var a;

a; // undefined
b; // ReferenceError: b is not defined
```

An annoying confusion is the error message that browsers assign to this condition. As you can see, the message is "b is not defined," which is of course very easy and reasonable to confuse with "b is undefined." Yet again, "undefined" and "is not defined" are very different things. It'd be nice if the browsers said something like "b is not found" or "b is not declared," to reduce the confusion!

There's also a special behavior associated with `typeof` as it relates to undeclared variables that even further reinforces the confusion. Consider:

```js
var a;

typeof a; // "undefined"

typeof b; // "undefined"
```

The `typeof` operator returns `"undefined"` even for "undeclared" (or "not defined") variables. Notice that there was no error thrown when we executed `typeof b`, even though `b` is an undeclared variable. This is a special safety guard in the behavior of `typeof`.

Similar to above, it would have been nice if `typeof` used with an undeclared variable returned "undeclared" instead of conflating the result value with the different "undefined" case.

### `typeof` Undeclared

Nevertheless, this safety guard is a useful feature when dealing with JavaScript in the browser, where multiple script files can load variables into the shared global namespace.

**Note:** Many developers believe there should never be any variables in the global namespace, and that everything should be contained in modules and private/separate namespaces. This is great in theory but nearly impossible in practicality; still it's a good goal to strive toward! Fortunately, ES6 added first-class support for modules, which will eventually make that much more practical.

As a simple example, imagine having a "debug mode" in your program that is controlled by a global variable (flag) called `DEBUG`. You'd want to check if that variable was declared before performing a debug task like logging a message to the console. A top-level global `var DEBUG = true` declaration would only be included in a "debug.js" file, which you only load into the browser when you're in development/testing, but not in production.

However, you have to take care in how you check for the global `DEBUG` variable in the rest of your application code, so that you don't throw a `ReferenceError`. The safety guard on `typeof` is our friend in this case.

```js
// oops, this would throw an error!
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}
```

This sort of check is useful even if you're not dealing with user-defined variables (like `DEBUG`). If you are doing a feature check for a built-in API, you may also find it helpful to check without throwing an error:

```js
if (typeof atob === "undefined") {
	atob = function() { /*..*/ };
}
```

**Note:** If you're defining a "polyfill" for a feature if it doesn't already exist, you probably want to avoid using `var` to make the `atob` declaration. If you declare `var atob` inside the `if` statement, this declaration is hoisted (see the *Scope & Closures* title of this series) to the top of the scope, even if the `if` condition doesn't pass (because the global `atob` already exists!). In some browsers and for some special types of global built-in variables (often called "host objects"), this duplicate declaration may throw an error. Omitting the `var` prevents this hoisted declaration.

Another way of doing these checks against global variables but without the safety guard feature of `typeof` is to observe that all global variables are also properties of the global object, which in the browser is basically the `window` object. So, the above checks could have been done (quite safely) as:

```js
if (window.DEBUG) {
	// ..
}

if (!window.atob) {
	// ..
}
```

Unlike referencing undeclared variables, there is no `ReferenceError` thrown if you try to access an object property (even on the global `window` object) that doesn't exist.

On the other hand, manually referencing the global variable with a `window` reference is something some developers prefer to avoid, especially if your code needs to run in multiple JS environments (not just browsers, but server-side node.js, for instance), where the global variable may not always be called `window`.

Technically, this safety guard on `typeof` is useful even if you're not using global variables, though these circumstances are less common, and some developers may find this design approach less desirable. Imagine a utility function that you want others to copy-and-paste into their programs or modules, in which you want to check to see if the including program has defined a certain variable (so that you can use it) or not:

```js
function doSomethingCool() {
	var helper =
		(typeof FeatureXYZ !== "undefined") ?
		FeatureXYZ :
		function() { /*.. default feature ..*/ };

	var val = helper();
	// ..
}
```

`doSomethingCool()` tests for a variable called `FeatureXYZ`, and if found, uses it, but if not, uses its own. Now, if someone includes this utility into their module/program, it safely checks if they've defined `FeatureXYZ` or not:

```js
// an IIFE (see "Immediately Invoked Function Expressions"
// discussion in the *Scope & Closures* title of this series)
(function(){
	function FeatureXYZ() { /*.. my XYZ feature ..*/ }

	// include `doSomethingCool(..)`
	function doSomethingCool() {
		var helper =
			(typeof FeatureXYZ !== "undefined") ?
			FeatureXYZ :
			function() { /*.. default feature ..*/ };

		var val = helper();
		// ..
	}

	doSomethingCool();
})();
```

Here, `FeatureXYZ` is not at all a global variable, but we're still using the safety guard of `typeof` to make it safe to check for. And importantly, here there is *no* object we can use (like we did for global variables with `window.___`) to make the check, so `typeof` is quite helpful.

Other developers would prefer a design pattern called "dependency injection," where instead of `doSomethingCool()` inspecting implicitly for `FeatureXYZ` to be defined outside/around it, it would need to have the dependency explicitly passed in, like:

```js
function doSomethingCool(FeatureXYZ) {
	var helper = FeatureXYZ ||
		function() { /*.. default feature ..*/ };

	var val = helper();
	// ..
}
```

There's lots of options when designing such functionality. No one pattern here is "correct" or "wrong" -- there are various tradeoffs to each approach. But overall, it's nice that the `typeof` undeclared safety guard gives us more options.

## Review

JavaScript has seven built-in *types*: `null`, `undefined`,  `boolean`, `number`, `string`, `object`, `symbol`. They can be identified by the `typeof` operator.

Variables don't have types, but the values in them do. These types define intrinsic behavior of the values.

Many developers will assume "undefined" and "undeclared" are roughly the same thing, but in JavaScript, they're quite different. `undefined` is a value that a declared variable can hold. "Undeclared" means a variable has never been declared.

JavaScript unfortunately kind of conflates these two terms, not only in its error messages ("ReferenceError: a is not defined") but also in the return values of `typeof`, which is `"undefined"` for both cases.

However, the safety guard (preventing an error) on `typeof` when used against an undeclared variable can be helpful in certain cases.

Because JavaScript is a very dynamic language, it can lend itself to many different coding styles and paradigms. However, some paradigms are more maintainable or performant than others. The following is a very opinionated guide based on my personal experience maintaining large or long-running JS projects.

__Table of Contents__

* [Style Guide](#style-guide)
    * [Braces](#braces)
    * [Semicolons](#semicolons)
    * [Indentation](#indentation)
    * [Context Closures](#context-closures)
    * [Quotes](#quotes)
    * [Equivalence](#equivalence)
    * [File Template](#file-template)
    * [Naming](#naming)
    * [Variable Declaration](#variable-declaration)
    * [Function Declaration](#function-declaration)
    * [Module Exports](#module-exports)
    * [Require Statements](#require-statements)
    * [Getters and Setters](#getters-and-setters)
* [Code Validation](#code-validation)
* [Third Party](#third-party)
* [Performance](#performance)
    * [Iterations](#iterations)
    * [Conversions](#conversions)
    * [Additional Reading](#additional-reading)
* [Workflow](#workflow)
    * [IDE](#ide)
    * [When to use a git branch](#when-to-use-a-git-branch)

## Style Guide

### Braces

Braces for code blocks generally get their own line, although closing braces are sometimes combined a closing parenthesis and semicolon. For literal object notation, the opening brace is placed on the same line as the assignment operator or return statement.

```javascript
if (test)
{
	//do something
}
else if (test2)
{
	//do something
}
else
{
	//do something
}

switch (item)
{
	case 1:
		//do something
		break;
	default:
		//do something
		break;
}

do
{
	//something
}
while (true);

var obj = {
	x: 1,
	y: 2,
	z: 3
};

function func ()
{
	return {
		x: 1,
		y: 2,
		z: 3
	};
}

eventEmitter.on('error', function (error)
{
	// ...
});
```

Occasionally it's nice to omit the braces for very simple if statements, though people will have mixed feelings about this. Certainly you should not mix braces and no-braces within a single control flow (in other words, if the _if_ statement has braces, the _else_ should too, even if it could omit them).

```javascript
if (false)
	return;
```

Simple one-line functions are also okay:

```js
function seven () { return 7; }
```

### Semicolons

Just use them. Seriously, what are you, lazy?

### Indentation

Use ___tabs___ for _logical_ indentation, not spaces. In other words, tabs are used to denote code blocks. If you need to align text to specific columns beyond the code block indentation level, use spaces because that is not _logical_ indentation, that's actually visual markup.

```javascript
function x ()
{
	// tab-indent to the code level

	//      use     spaces    to   line   up   comments if necessary
	//          1    2    3      4     5
	function y (one, two, three, four, five) {}
}
```

The typical JavaScript style uses two spaces for indentation. This simply isn't enough for me visually to line up blocks down the page at a glance, and therefore requires more mental effort to parse the code. Other people prefer this more condensed look. Tabs allows everyone to get the visual style they want without forcing their own preference on others.

### Context Closures

Because the [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) keyword is an odd little thing, it is often necessary to create an alias. ___Always___ name that alias variable `_this`. Some people use `self` or `that`, but the great thing about `_this` is that if you do a `ctrl-f` for `this.*`, it will always find what you were looking for regardless of whether the alias is being used.

```javascript
Class.prototype.funcA = function ()
{
	var _this = this;
	function innerFunction ()
	{
		var x = _this.x * 7;
		//...
	}
	//...
};
```

### Quotes

For string literals, single-quotes `'` are preferred over double-quotes `"` primarily because hitting the shift key is hard. If your string needs to use literal single-quotes inside it, feel free to use double-quotes instead.

```javascript
var str1 = 'basic string';
var str2 = "It's annoying to have to escape the ' character, don't you think?";
```

### Equivalence

You should almost always use `===` instead of `==`. There are a few limited instances where `==` may be desirable such as when comparing a number to a value which might not be a number, but may have an equivalent value. Even in that case, it may be better to convert the value to a number first. If you do use `==`, make sure you understand the consequences and have a good reason for doing so. You'll also have to disable the JsHint check for that block using inline configuration `/* jshint eqeqeq: false */`

```javascript
if (a === b) // good
if (a == b)  // generally bad
```

### File Template

I use the following template for most non-static JavaScript modules:

```
"use strict";
/* -------------------------------------------------------------------
 * Require Statements << Keep in alphabetical order >>
 * ---------------------------------------------------------------- */

//

/* =============================================================================
 * 
 * ModuleName - DESCRIPTION
 *  
 * ========================================================================== */

module.exports = ModuleName;

function ModuleName ()
{
}

/* -------------------------------------------------------------------
 * Getters / Setters
 * ---------------------------------------------------------------- */

//

/* -------------------------------------------------------------------
 * Static Methods << Keep in alphabetical order >>
 * ---------------------------------------------------------------- */

//

/* -------------------------------------------------------------------
 * Prototype Methods << Keep in alphabetical order >>
 * ---------------------------------------------------------------- */

//

/* -------------------------------------------------------------------
 * Private Methods << Keep in alphabetical order >>
 * ---------------------------------------------------------------- */

//

```

Static modules follow a similar format, but don't typically include a constructor (use `var ModuleName = module.exports` instead) or the Prototype Methods section.

Not all of the sections will be applicable for each new module, so feel free to edit or delete sections as necessary. Re-ordering of sections should be kept to a minimum, and you should have an actual reason for doing so.

As you can tell, I prefer to keep methods in alphabetical order. This is more of a preference than a requirement, but it helps you know which direction to scroll to find a method without having to search. Developers tend to group related functions together, which seems reasonable, except that the overall order on the page becomes chaos. What I propose is that you order all methods alphabetically excluding any common prefixes (like get/set/update/etc...). This way related functions will still be grouped, and you'll always know the right place to put functions and look for them.

If you decide the template does not apply at all to your file, at a minimum, keep the `"use strict";` at the very top of the file. Doing so ensures our code will be more future-proof, makes certain errors more apparent, and enables harmony features. For a more complete description of strict mode, see: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode

### Naming

`TitleCasing` for constructors (functions you would call with the `new` keyword) and namespaces, uppercase `SNAKE_CASING` for constants, and `camelCasing` for everything else.

```javascript
var THIS_SHOULD_NOT_BE_CHANGED = "don't do it";

function MyClass (x)
{
	this.member = x;
}

function myFunction (x)
{
	return x + 1;
}

var myObj = new MyClass(1);
```

File names should use `TitleCasing` if they export a an object, namespace, or constructor. If a file exports a function, it should use `camelCasing`.

### Variable Declaration

There is no need to declare all of your variables at the top of a function. In fact, it is preferred to declare them where it is most logically relevant (such as just before its first use or assignment) in order to improve code readability.

__Combining declarations__

Anytime variable declaration is combined with an assignment, the declaration should not be combined with other variables. However, if multiple variables are being declared, but not assigned, they can, and probably should, be combined into a single declaration on one line.

```javascript
var a, b, c;
var d = 8;
var e = 'e';
```

The same is true for `require()` statements.

```javascript
var ModuleA = require('ModuleA');
var ModuleB = require('ModuleB');
var ModuleC = require('ModuleC');
```

This makes it very easy to add and remove variables, and no matter where you add or remove them, it will always create a one-line diff.

__var vs. let__

If you run node with the `--harmony` flag, you can use certain ECMAScript6 features, such as [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let). Using `let` is a great idea... except there's one big problem: the V8 optimizer does not support it yet. This means the code will compile and run correctly, but will usually cause the entire function which encloses it to run slower (often a 2x performance difference). So, until V8 optimizes block-scoping, you should avoid using let/const in performance-critical code paths. However, generator functions are currently not optimized either, so feel free to use let/const inside generator functions without any additional performance penalties.

### Function Declaration

For non-member functions (functions which are not attached to an object or prototype), __named functions are preferred__ over function pointers. The reason is because named functions can be called from anywhere inside the same scope, but function pointers can only be called after the point in code where the pointer is assigned.

```javascript
funcA(); // <-- this will work because it's a named function
funcB(); // <-- this will not work because funcB is undefined

function funcA () {}
var funcB = function () {}; // try not to do this much
```

Remember that named functions cannot be created in inner scopes (such as the inside of an if statement) according to the ES6 standard. V8 is not yet enforcing this, but it may eventually, and we should avoid doing so.

```javascript
function myFunction ()
{
	function nestedInsideFunctionBlock () {} // OK

	if (test)
	{
		function nestedInsideInnerBlock () {} // BAD
	}
}
```

### Module Exports

Don't use `module.exports` more than once in a module, and ___never___ use the `exports` shortcut. Instead, create an alias or use a constructor.

For example, a module of static methods might do this:

```javascript
var MyModule = module.exports;
MyModule.funcA = function () {};
```

Or a constructor-style module might do this:

```javascript
module.exports = MyConstructor;
function MyConstructor (x)
{
	this.x = x;
}

MyConstructor.prototype.funcA = function () {};
```

The alias name should be the name of the file (minus `.js`) and the same as what you would call the module when requiring it from elsewhere.

```javascript
var MyModule = require('MyModule');
var MyConstructor = require('MyConstructor');
```

### Require Statements

Because a require statement generally returns a constructor or a namespace, it is preferred to use TitleCasing for most imported dependencies. The added benefit of using TitleCasing instead of camelCasing is that it helps prevent name collisions with local variables or arguments. The only time the imported variable name should be lowercase is when what you are importing is a function (this was JSHint doesn't complain that you're trying to call a constructor without `new`).

For example, you may want to use the argument name `path` in a function, but also want to call methods from the `path` module. Changing the require statement to `var Path = require('path');` facilitates both in a very intuitive and non-disruptive way.

```javascript
var ChildProcess = require('child_process');
var Fs = require('fs');
var Path = require('path');
```

__Also:__ keep require statements in alphabetical order. It simply makes it a lot easier to see, at a glance, if the dependency you're looking for has already been included.

### Getters and Setters

JavaScript has several ways of defining getters and setters. Until the simple `get` and `set` syntax is optimized in V8, use [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) or [Object.defineProperties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) to create getter and setter properties. `defineProperty` is preferred over `defineProperties` because it gives better autocomplete in WebStorm.

```javascript
function MyClass (miles)
{
	this.miles = miles;
}

Object.defineProperty(MyClass.prototype, 'nauticalMiles',
{
	get: function () { return this.miles / 1.15; }
});

var x = new MyClass(7);
console.log(x.nauticalMiles); // 6.086956521739131
```

## Code Validation

You should consider using [JSHint](http://www.jshint.com/) and other tools to perform static analysis before deploying code.

## Third Party

Third party tools are great. It would take us forever to write anything if we weren't building off the shoulders of others. There are a few points to keep in mind when considering whether to include third party modules, and how to use them.

### Before Including

First, consider whether the module really adds functionality value, or is more equivalent to syntactic sugar. If it's the latter, be wary of how it will impact performance. In many cases, it is simply not worth including. On the other hand, if it solves a problem for you in less time than it would take for you to write your own solution, go for it.

We should try to remember to check the license before including a new dependency, although this can be a little bit difficult considering the nested dependencies. Luckily, almost everything in the Node.js world is MIT/BSD/Apache licensed, which is great for us.

### Including

Always remember to use the `--save` flag when npm installing so that the dependency is automatically added to package.json.

### Usage

__Object Wrapping__

Don't use third party libraries to wrap built-in objects. JavaScript's built-in `Date` object works great for almost everything you could need to do with dates, so don't use something like `moment()`. When a function specifies an argument like `startDate` it should be a `Date` object, or a number at the very least, not a `moment` object. However, what moment _is_ good at is formatting the output of a date. So if you have a need to output a date in a particular format, feel free to use moment for that, just don't pass the object around pretending it's a Date.

__Syntax Sugar__

Try not to get carried away with using libraries like [underscore](http://underscorejs.org/) for syntax sugar. Partly for readability, partly for performance, and partly because there are tools built-in to JavaScript to do the same things, only faster than the library can. Remember that, for server-side JavaScript, there is absolutely no need to worry about browser compatibility issues.

For example, don't use `_.keys(obj)`. Just use `Object.keys(obj)`.

On the other hand, `_.sample(arr, 6)` doesn't have a built-in counterpart, and may be a quick and readable way to sample elements from an array.

__Moral of the story__

Get to know the [Standard built-in objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects) and the [Node.js standard library](http://nodejs.org/api/) if you're not already pretty familiar with them. They can often do more than people realize at first.

## Performance

This section provides some tips for writing more performant code.

### Iterations

__for...in__

[for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) should generally not be used for Arrays, but is typically okay for objects. Keep in mind that for...in iterates over enumerable properties in the object's prototype, which may produce unexpected results. To only iterate over an object's own properties, use `Object.keys()` and then iterate over the returned array. This is faster than checking `.hasOwnProperty()` on each iteration.

```javascript
var ownProps = Object.keys(obj);
for (var i = 0; i < ownProps.length; i++)
{
	doSomething(obj[ownProps[i]]);
}
```

__for vs. forEach__

[Array#forEach](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) should usually be avoided. A good old fashion `for` loop is an order of magnitude faster and often easier to read. Even when all the loop does is call a function on every iteration, it is still drastically faster than the `forEach` method.

```js
for (var i = 0; i < arr.length; i++)
{
	doSomething(arr[i]);
}
```

If you want a very slight performance improvement, on really long arrays, cache the `arr.length` into a local variable, and then use that in the loop condition instead.

[jsperf: for vs. forEach](http://jsperf.com/se-for-vs-foreach)

__Array Iterator Functions__

What is true about `Array#forEach` is true for all of the Array iterator functions, such as [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map). Consider that this code:

```javascript
var copy = new Array(items.length);
for (var i = 0; i < items.length; i++)
	copy[i] = items[i];
```

is orders of magnitude faster than:

```javascript
var copy = items.map(function (item) { return item; });
```

See: [jsperf: map vs for loop](http://jsperf.com/se-map-vs-for-loop)

This may not be a big deal when dealing with arrays with only a handful of elements in a code path which isn't executed repeatedly on every request. However, if there is any concern that the code path may produce a bottleneck, simply use a loop instead.

__TL;DR__

Good old fashion for loops are great. Any other form of iterating will _always_ hurt performance to some degree.

### Conversions

__Booleans__

`!!val` is faster than `Boolean(val)`. [jsperf: !! vs. Boolean](http://jsperf.com/se-not-not-vs-boolean)

__Numbers__

`Number(val)` is faster than `+val` unless val is a string literal, or a local variable which was initialized from a string variable... but why would you ever create a string literal and then convert it back to a number? [jsperf: + vs Number](http://jsperf.com/se-plus-vs-number)

`val|0` is marginally faster than `Math.floor(val)`. For large numbers, `Math.floor()` is safer.

__Strings__

`val+''` is faster than `val.toString()` or `String(val)`, however, it may have different behavior on objects which have both `valueOf()` and `toString()` methods defined. If there is any doubt about what type of object it is, `String(val)` is safer.

### Additional Reading

* [Optimization Killers](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)

    Describes many of the scenarios which will prevent V8 from optimizing a function.
* [Performance Tips for JavaScript in V8](http://www.html5rocks.com/en/tutorials/speed/v8/)

    Describes how V8 uses "hidden classes" and what kinds of behavior hurt that optimization. It also links to a great talk by [Daniel Clifford at Google I/O 2012](http://www.youtube.com/watch?v=UJPdhx5zTaw) which covers much of the same ground as the article, but is well worth the watch if you have time.

## Workflow

### IDE

If you prefer to work in an IDE, [WebStorm](http://www.jetbrains.com/webstorm/) is recommended. You can download a 30-day trial, and a license is around $50.

* Code completion
* Integrated debugging
* Git integration
* Code inspection

If you do use WebStorm, consider downloading my pre-made [WebStormSettings.jar](https://github.com/bretcope/webstorm-settings) and importing some, or all, of the settings. Go to `File->Import Settings` and it will give you a window with several checkboxes allowing you to selectively import settings from the jar. In particular, if you want to use my codeing style, you should import the "Code style schemes" setting. After import, go to `File->Settings->Code Style` and select `Scheme: Bret` and apply. It also has a preset called `General` which is more similar to the typical JS style (two spaces, opening braces on the same line, etc) which may be useful for other projects.

#### Debugging Node 0.11.x

There is a bug in 0.11.13 (which will be fixed in the next version) relating to inspecting globals. WebStorm, by default, tries to read globals when it starts debugging a process. This causes it to crash. The work-around is to tell WebStorm to not automatically load globals. Go to `IDE_INSTALLATION_DIR/bin` and edit the file ending in `.vmoptions`. Add the line:

    -Djs.debugger.load.global.variables.on.start=false

Restart WebStorm for the change to take effect.

### When to use a git branch

When working on a team where everyone has push access, most work should be done on the __master__ branch, and rebase is preferred over merging (`git pull --rebase`). Feature flags are preferred over branching. Separate branches should be used when a new feature or change is experimental, but only if it is unlikely to cause merge conflicts. Refactoring of existing code should ___never___ be done on a branch other than master. It is your responsibility to make sure your branch stays up to date with master during development (use `git pull origin master` to merge changes from master into your branch).

Pull requests are a great way to do code reviews, however, there are other ways of asking for code reviews for simple things, such as drop a link to the diff in chat before deploying production. So don't feel like a PR is the only way to get a second set of eyes on something.
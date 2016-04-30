---
title: Advanced JavaScript -- Scope
category: JavaScript
tags: JavaScript
---

> notes from on-line course: [Advanced JavasSript](https://app.pluralsight.com/library/courses/advanced-javascript/table-of-contents) by [*Kyle Simpson*](http://blog.getify.com/)


# Where to look for things

JavaScript has **function scope** only.

```javascript
var foo = "bar";

function bar() {
    var foo = "baz";

    function baz(foo) {
        foo = "bam";
        bam = "yay";
    }
    baz();
}

bar();
foo; // bar
baz(); // Error
```

## Function expression

In named function expression, the variable `bar` is declared inside function `foo`, it's not accessible from outside.

```javascript
var foo = function bar() {
    var foo = 'baz';
    function baz(foo) {
        foo = bar;
        foo; // function...
    }
    baz();
}
foo();
bar(); //Error
```
named function expression has 3 advantages than anonymous functions

- it has a way to reference itself inside function, because `bar` is in `bar` function scope, not global scope. `bar` is self while `this` is global
- useful for debug
- serves as documentation of itself


## Block scope?

`catch` will create a block scope.

```javascript
var foo;

try {
    foo.length;
}
catch (err) {
    console.log(err); // TypeError, foo.length
}
console.log(err); // ReferenceError
```

## Lexical scope, dynamic scope

Lexical scope is compile time scope. Variables first look in current scope, if not found go one scope above, if not even in global scope, throw `ReferenceError`.

```javascript
// inside global scope
function foo() {
// inside function foo scope
    var bar = "bar";

    function baz() {
// inside function baz scope
        console.log(bar); // bar
    }
    baz();
}
foo();
```

There're serveral ways to cheat scope by `eval` and `with`. But they are problematic.

```javascript
var bar = 'bar';
function foo(str) {
    eval(str);
    console.log(bar); // 42
}
foo('var bar = 42;');
```

```javascript
var obj = {
    a: 2,
    b: 3,
    c: 4
};

obj.a = obj.b + obj.c;
obj.c = obj.b - obj.a;

with (obj) {
    a = b + c;
    c = b - a;
    d = 3; // ??
}

obj.d; // undefined
d; // 3
```

## IIFE

Immediately Invoked Function Expression(IIFE) creates an scope that encloses all the variables and functions inside. It could also accepts variables.

```javascript
var foo = '1';

(function(bar) {
    var foo = bar;
    console.log(foo); // 1
})(foo);

console.log(foo); // 1
```

## ES6+ let

The `let` keyword in ES6 with curly brackets will create a new scope. `var` will bind the declaration in function, `let` will bind the declaration in brackets.

However, there is a problem with `let` keyword:

It will NOT *hoist* the declaration, meaning that in this scope, trying to use the variable before it's declaration throws an error. So it need to put at top of scope if necessary.

```javascript
var foo = 'bar';
{
    let foo = 1;
}
foo; // bar
```

## Quiz

Q: What type of scoping rules does JavaScript have? Exceptions?

A: Lexical scoping. `eval` and `with` will cheat.

Q: What are the different ways to create a scope?

A: By `function` declaration, by `catch` block or by `let` keyword.

Q: What's the difference between *undeclared* and *undefined*?

A: undefined is variable that is declared but not assigned value.


## hoisting

```javascript
a; //undefined
b; //undefined
var a = b;
var b = 2;
b; // 2
a; // undefined, copy by b's value
```

is equivalent to 

```javascript
var a;
var b;
a = b;
b = 2;
b;
a;
```
which means declarations (variable and function) is hoisted to the top of scope. Further more, function declarations come before variable declarations.
If variable and function has the same name, variable will be overwritten by function declaration.

```javascript
var a = b();
var c = d();
a;  //?
c;  //?

function b() {
    return c;
}

var d = function () {
    return b();
}
```
is equivalent to 

```javascript
function b() {
    return c;
}
var a;
var c;
var d;
a = b();
c = d();
a; // undefined
c; // TypeError: d is not a function
d = function() {
    return b();
}
```

## this keyword

Every function, while executing, has a reference to its current executing context called `this`. For Lexical scope vs dynamic scope, JavaScript's dynamic scope is `this`.

There's are 4 level top-down rules to determine what `this` points to, they are:

1. `new`, is this function called with `new`? if so, `this` points to the created object.
2. explicit, is this function called with `bind` or `call` or `apply`, if so, `this` points to the given object.
3. implicit, is this function called via a containing/owning object(context)? if so, `this` points to the object/context.
4. default, global object (undefined in strict mode).

To determine if it falls into 3 or 4, it requires to look at the **call site**.
Consider this example:

```javascript
var o1 = {
    bar: "bar1",
    foo: function() {
        console.log(this.bar);
    }
};
var o2 = { bar: "bar2", foo: o1.foo };

var bar = "bar3";
var foo = o1.foo;

o1.foo();       // "bar1"
o2.foo();       // "bar2"
foo();          // "bar3"
```
`foo()` returns 'bar3' because the call site is `global.foo()`, `this` points to global, so `this.bar` means `global.bar` which is `bar3`.




















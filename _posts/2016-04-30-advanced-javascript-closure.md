---
title: Advanced JavaScript -- Closure
category: JavaScript
tags: JavaScript
---

> notes from on-line course: [Advanced JavasSript](https://app.pluralsight.com/library/courses/advanced-javascript/table-of-contents) by [*Kyle Simpson*](http://blog.getify.com/)

**Closure is when a function "remember" its lexical scope even when the function is excuted outside that lexical scope.**

```javascript
function foo() {
    var bar = "bar";

    return function baz() {
        console.log(bar);
    }
}
function bam() {
    foo()(); // bar
}
bam();

```

another example:

```javascript
function foo() {
    var bar = "bar";

    $('btn').click(function(e) {
        console.log(bar);
    });
}

foo();
```

shared closure:

```javascript
function foo() {
    var bar = 0;

    setTimeout(function() {
        console.log(++bar);
    }, 1000)

    setTimeout(function() {
        console.log(++bar);
    }, 2000)

}

foo(); // 1,2
```

## Common problem

It runs into problem when used closure in loops. Consider the code:

```javascript
for(var i=0; i<10; i++) {
    setTimeout(function() {
        console.log(i);
    }, 100*i);
}
```
It will output 10 ten times because it refer to the same `i` that is 10 when timeout function executes.

To fix it, pass different value for each timeout function.

```javascript
for(let i=0; i<10; i++) {
    setTimeout(function() {
        console.log(i);
    }, 100*i);
}
```
`let` will create different `i` for each loop.

another solution:

```javascript
for(var i=0; i<10; i++) {
    (function(i){
        setTimeout(function() {
            console.log(i);
        }, 100*i);
    })(i)
}
```
This solves the problem because JavaScript copy primitive values when pass arguments in. If you try this, it won't work.

```javascript
var obj = {};
for(obj.i=0; obj.i<10; obj.i++) {
    (function(o){
        setTimeout(function() {
            console.log(o.i);
        }, 100*o.i);
    })(obj)
}
```
Because JavaScript pass `obj` by copy a reference to it, so every timeout function still points to the same `obj` with different reference to it. 

> [Primitives are passed by value, Objects are passed by "copy of a reference".](http://stackoverflow.com/questions/13104494/does-javascript-pass-by-reference).

Note that by definition, this doesn't create a closure because it's not a function but a object.

```javascript
var foo = (function() {
    var o = {bar: 'bar'};
    return {obj: o};
})();

console.log(foo.obj.bar); // 'bar'
```

## Module pattern

It's easy to create a module over closure. Just do 2 things:

1. declare a function to create a lexical scope
2. return one or more inner functions that have a closure over the scope.

all the implementation details are hidden in private functions inside main function.

for example: 

```javascript
var module = (function () {
    var bar = 'bar'; //private
    var api = {
        foo: function() {
            console.log(bar);
        },
        baz: function() {
            api.foo();
        }
    };
    return api;
})();

module.foo(); // 'bar'
```

some modern module libraries are also derived from this module pattern, like *RequireJS*. The syntax is similar:

```javascript
define('someModule', function() {
    var foo = "foo";

    return {
        bar: function() {
            console.log(foo);
        }
    }
})
```

Or ES6 module pattern:

```javascript
// in foo.js
var o = {bar: 'bar'};

export function bar() {
    return o.bar
}

// in bar.js
import bar from 'foo';
bar();

module foo from 'foo';
foo.bar(); // 'bar'

```

They are fundamentally the same mechanism except that libraries do the wrapper.

## Quiz

Q: How is closure created?

A: When a inner function is transported outside of outer function.

Q: How long does its scope stay around?

A: A closure is kinda like a reference to a hidden scope object, as long as some function still has closure over its scope, that scope's going to stay around. But as soon as the closure goes away, the scope can be garbage collected.

Q: Why doesn't a function callback inside a loop behave like expected? how to fix it?

A: There wasn't a variable created per iteration.

Q: what's the benefits of the module pattern? What's the trade off?

A: The least exposure principle, hiding things, have public APIs. Some people believe that module pattern make it hard or impossible to test private functions. Another trade off may be when 1000 modules are created, it has 1000 copies of that modules inner functions and APIs.




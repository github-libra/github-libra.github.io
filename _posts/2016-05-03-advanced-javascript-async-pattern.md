---
title: Advanced JavasScript -- Async Pattern
category: JavaScript
tags: JavaScript
---

> notes from on-line course: [Advanced JavasSript](https://app.pluralsight.com/library/courses/advanced-javascript/table-of-contents) by [*Kyle Simpson*](http://blog.getify.com/)

## Callbacks

Async can be achieved by callbacks.

```javascript
setTimeout(function callback() {
    console.log('time up');
}, 1000)
```

But it often results in *Callback Hell*.

```javascript
setTimeout(function callback() {
    console.log('foo');
    setTimeout(function callback() {
        console.log('bar');
        setTimeout(function callback() {
            console.log('baz');
        }, 1000)
    }, 1000)
}, 1000)
```

*Callback Hell* doesn't just in its forms, above code can also be written like this:

```javascript
function one(cb) {
    console.log('foo');
    setTimeout(cb, 1000);
}
function two(cb) {
    console.log('bar');
    setTimeout(cb, 1000);
}
function three(cb) {
    console.log('baz');
    setTimeout(cb, 1000);
}

one(function() {
    two(three)
});
```

Essentially callbacks gives the *control* over to somebody else, it's up to other piece of code when or how or not to call you callback, that's **Inversion of Control**.

Multiple callbacks can be passed as arguments, and in NodeJS it's a convention to pass error as first argument in callback.

## Generators(yield)

The `yield` keyword causes generator function execution to pause and the value of the expression following the yield keyword is returned to the generator's caller. It can be thought of as a generator-based version of the `return` keyword.

The yield keyword actually returns an `IteratorResult` object with two properties, value and done. The value property is the result of evaluating the yield expression, and done is `false`, indicating that the generator function has not fully completed.

Once paused on a yield expression, the generator's code execution remains paused until the generator's `next()` method is called. Each time the generator's `next()` method is called, the generator resumes execution


## Promises

Promises can be treated as *continuation events*. It's like initiating an async action, instead of getting results right away, it returns a promise. And you can set up `success` and `fail` listener on some events triggered some time later. It gives the **control** back to the caller to decide what to to next.  

```javascript
//jQuery style promise
function wait(n) {
    var p = $.Deferred();
    setTimeout(function() {p.resolve()}, n);
    return p.promise();
}

wait(1000)
    .then(function() {
        console.log('hello');
        return wait(2000);
    })
    .then(function() {
        console.log('world');
    });
``` 

```javascript
//native Promise
function getData(n) {
    return new Promise(function(resolve, reject) {
        setTimeout(function() { resolve(n)}, 1000);
    });
}

var x = 1;
getData(10)
    .then(function(n1) {
        x = n1 + 1;
        return getData(x);
    })
    .then(function(n2) {
        x = n2 + 1;
        return getData('meaning of life' + (x + y));
    })
    .then(function(answer) {
        console.log(answer);
    });
```

Native Promise API seems tedious to create promises when doing real-world complex flows. [ASQ](https://github.com/getify/asynquence) is designed to solve it. It by default return `promise` and support features like `gate`, `seq`, `waterfall`.

```javascript
ASQ()
    .then(function(done) {
        setTimeout(done, 1000);
    })
    .gate(function(done) {
        setTimeout(done, 1000);
        setTimeout(done, 1000);
    })
    .then(function() {
        console.log('2 seconds passed');
    });

```



























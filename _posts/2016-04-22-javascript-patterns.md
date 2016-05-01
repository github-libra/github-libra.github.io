---
title: Common JavaScript Patterns
category: JavaScript
tags: JavaScript, pattern
---

> notes from on-line course [Javascript Design Patterns](https://app.pluralsight.com/library/courses/javascript-design-patterns/table-of-contents) by [Aaron Powell](http://www.aaron-powell.com/)

## Function arguments

Java has function overloading to provide same functionality with different arguments. JavaScript can do this by `arguments` inside every function.

`arguments` is an *array-like* object which means `arguments[0]` will return the first argument. And Array functions can be performed.

```javascript
function getArgumentsInArray() {
    return Array.prototype.slice.call(arguments);
}
getArgumentsInArray(1,2,3); //output: [1,2,3]

```


## Chainable

- return `this` on each function call
- keep changing some property of one object

```javascript
function Calc(num) {
    this.add = function(a) {
        num += a;
        return this;
    }
    this.minus = function(a) {
        num -= a;
        return this;
    }
    this.equals = function(callback) {
        callback(num);
        return this;
    }
}
new Calc(5)
    .add(1)
    .minus(2)
    .equals(function(v) { 
        // v = 4;
    })
.
```


## Method of properties

In ES5 Object has a `defineProperty()` which will allow bind functions when trying to `get` or `set` the property.
eg: 

```javascript
var tom = {};
Object.defineProperty(tom, 'age', {
    get: function() {return 20;},
    set: function(v) { console.log('trying to set age to ', v); }
})
tom.age = 21; // output: trying to set age to 21
tom.age // = 20;

// another way
tom = {
    get age() {return 20},
    set age(v) { console.log('trying to set age to ', v); }
}
```

Code below is trying to achieve the similar functionality.

```javascript
function Book(name, price) {
    var priceChanging = [],
        priceChanged = [];

    this.name = function() {
        return name;
    }
    this.price = function() {
        return price;
    }

    this.setPrice = function(value) {
        var i = j = 0;
        if(value && value !== price) {
            for(; i<priceChanging.length; ++i) {
                if(!priceChanging[i](this, value)) {
                    return value;
                }
            }
        }
        price = value;

        for(; j<priceChanged.length; ++j) {
            priceChanged[j](this);
        }
        return this;
    }

    this.onPriceChanging = function(callback) {
        priceChanging.push(callback);
    }

    this.onPriceChanged = function(callback) {
        priceChanged.push(callback);
    }
}

var book = new Book('Civil War', 20);
book.name();
book.onPriceChanging(function(book, price) {
    if(price > 100) {
        console.log('too high for a book');
        return false;
    }
    return true;
});

book.onPriceChanged(function(book) {
    console.log('price changed to $', book.price());
});

book.setPrice(50);
book.setPrice(110);
```

## Async execution pattern

Browsers are typically single threaded which means UI and JavaScript code cannot run simultaneously, which also means long running JavaScript code will cause UI jagging.

Splitting long running code over setTimeout blocks releases the thread. Ensure there's enough time gaps between setTimeouts restarting.

Below `buffer()` try to do as many `itemFn()` as possible in 50 ms, it one `itemFn()` lasts longer than 50 ms, it waits for ~20 ms to continue. Thus allow UI thread to have time react to user interaction.

```javascript
var buffer = function(items, itemFn, done) {
    var i = 0, len = items.length;

    setTimeout(function() {
        var result;

        for(start = +new Date(); i < len && result !== false && (+new Date() - start < 50); ++i) {
            result = itemFn.call(items[i], items[i], i);
        }

        if(i < len && result !== false) {
            setTimeout(arguments.callee, 20);
        }else {
            done(items);
        }

    }, 20);

}
```

## Recursive setTimeout pattern

Compare two following examples:

```javascript
// 1
setInterval(function API() {
    $.get('/data', function() {

    })
}, 1000);

// 2
setTimeout(function API() {
    $.get('/data', function() {
        setTimeout(API, 1000);
    })
}, 1000);
```
They're the same when `API()` doesn't cost any time to execute; 

- When `API()` takes 100 ms, 1 fires a request every 1000 ms, 2 fires requests every 1100 ms;
- WHen `API()` takes 2000 ms, 1 fires a request every 1000 ms, 2 fires requests every 3000 ms;

`setInterval` ordering is unpredictable over browsers, using recursive setTimeout pattern results in call in sequence.


## Async module definition (AMD)

JavaScript lacks ways to have external dependencies. 
- How to specify code blocks that rely on other code blocks?
- How to prevent scope leakage?
- How about NodeJs? it doesn't need to download file.

First, NodeJs uses CommonJS pattern, call `require` and `module` to import and export modules. When require a module, NodeJs wraps JS file in a function which makes functions/variables not accessed directly after required. It requires `module.exports` explicitly exports.

```javascript
//pi.js
module.exports = PI = 3.14;

//a.js
var pi = require('pi');
console.log(pi); //output: 3.14
```

RequireJs is a module framework that offers more browser-friendly API and supports NodeJS. It doesn't support CommonJS's module pattern.

First, define entry point by `data-main` attribute.

```html
<script data-main="main.js" src="require.js"></script>
```
in main.js, use `require()` to load dependencies.

```javascript
require(['jquery', './api'], function($, api) {
    $(document).ready(function() {
        api.getVersion();
    });
})
```

expose `api` in api.js file over `define`.

```javascript
define(['jquery'], function($) {
    return {
        getVersion: function() {
            return $.get('path/to/api');
        }
    }
})
```
RequireJs will loads all the dependent files automatically. RequireJs also allow to dynamically load scripts at runtime.

```javascript
define(['require'], function(require) {
    if(readyToGo) {
        require(['./car'], function(car) {
            car.ignite();
        })
    }else {
        require(['./tv'], function(tv) {
            tv.open();
        })
    }
})
```


## Sub/Pub pattern

In JavaScript a lot of actions are driven by events, like click/blur/change. There's Sub/Pub pattern in JavaScript logic to bridges the gap. A piece of code raises an event, N pieces of code react to it which allows for disconnected communication between functions.

A example using RequireJs would be:

```javascript
define(function() {
    var cache = {};

    return {
        pub: function(id) {
            var args = [].slice.call(arguments, 1);

            if(!cache[id]) {
                cache[id] = [];
            }
            for(var i=0; i<cache[id].length; ++i) {
                cache[id][i].apply(null, args);
            }
        },
        sub: function(id, fn) {
            if(!cache[id]) {
                cache[id] = [];
            }
            cache[id].push(fn);
        },
        unsub: function(id, fn) {
            if(!id) {
                return;
            }
            if(!fn) {
                cache[id] = [];
            }
            var i = cache[id].indexOf(fn);
            if(i > -1) {
                cache[id].splice(i, 1);
            }
        }
    }
});
```
This implementation would not cover one situation in which some subscribers won't be notified by earlier events. To do this, it requires to store the arguments from the `pub()`, call that in `sub()` which gives a bit of history.


## Promises and Deferred

- A deferred object controls operation
- Progress raised as deferred runs
- Done raised when deferred action runs successfully
- Fail raised when deferred actions is unsuccessful
- **Future access to the deferred result should not change**

A demo of Defer API would be:

```javascript
var Promise = function() {
    var data,
        fail = done = [],
        status = 'progress';
    this.done = function(fn) {
        done.push(fn);
        if(status === 'done') {
            fn(data);
        }
        return this;
    };
    this.failed = function(fn) {
        fail.push(fn);
        if(status === 'fail') {
            fn(data);
        }
        return this;
    };
    this.resolve = function(result) {
        if(status !== 'progress') {
            throw 'already resolved';
        }
        status = 'done';
        data = result;
        for(var i=0, il=done.length; i<il; ++i) {
            done[i](data);
        }
        return this;
    };
    this.fail = function(reason) {
        if(status !== 'progress') {
            throw 'already resolved';
        }
        status = 'fail';
        data = reason;
        for(var i=0, il=fail.length; i<il; ++i) {
            fail[i](data);
        }
        return this;
    };
}

```






















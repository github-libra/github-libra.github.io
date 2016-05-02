---
title: Advanced JavasScript -- OO
category: JavaScript
tags: JavaScript
---

> notes from on-line course: [Advanced JavasSript](https://app.pluralsight.com/library/courses/advanced-javascript/table-of-contents) by [*Kyle Simpson*](http://blog.getify.com/)

Every single object is built by a constructor function. Each time a constructor is called, a object is created. A constructor makes an object **linked** to its own **prototype**.

Consider this example:

```javascript

function Foo(who) {
    this.me = who;
}
Foo.prototype.identify = function() {
    return "I am " + this.me;
};

function Bar(who) {
    Foo.call(this,who);
}
// Bar.prototype = new Foo(); // OR...
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.constructor = Bar;

Bar.prototype.speak = function() {
    alert("Hello, " + this.identify() + ".");
};

var b1 = new Bar("b1");
var b2 = new Bar("b2");

b1.speak();
b2.speak();
```

If circle means function, square means object, the code above can be drew like this:
![code in picture](/images/code-in-picture.png);

So [[Prototype]] comes from an object linked to another object. It can be created by `object.create` or the 2nd phrase of `new` keyword. It affects an object in the way that if some properties or function references cannot be found in current object, it delegates to its [[Prototype]] object. And we can get an object's [[Prototype]] by `__proto__` property, or by `Object.getPrototypeOf()` or by `.constructor.prototype` property.


## OLOO pattern

OLOO means *Objects Linked to Other Objects*. To rewrite the above code in OLOO pattern, it will be like:

```javascript
var Foo = {
    init: function(who) {
        this.me = who;
    },
    identify: function() {
        return "I'm " + this.me;
    }
};

var Bar = Object.create(Foo);

Bar.speak = function() {
    alert('Hello, ' + this.identify());
}

var b1 = Object.create(Bar);
b1.init('b1');
var b2 = Object.create(Bar);
b2.init('b2');

b1.speak(); // Hello, I'm b1
b2.speak(); // Hello, I'm b2
```

It doesn't have any code of `new` or `prototype` or `constructor` but only `Object.create`  which is simpler. And it separate object creation from initialization. The code above resolves into another simpler diagram:

![code-in-oloo-picture](/images/code-in-oloo-picture.png)


## Inheritance

JavaScript has prototype inheritance, which is different from classical OO inheritance in that it doesn't copy properties and behaviors of constructor but links or delegates to it. This kind of inheritance(delegation) has some tradeoffs:

- It is more efficient from performce point of view.
- It is more dynamic and changing at runtime, while classical inheritance is like a snapshot of classes.
- Shadowing(override) is kind of awkward in JavaScript
- Everything is public without private state and encapsulation.

![classical inheritance](/images/classical-inheritance.png)
![prototype inheritance](/images/prototype-inheritance.png)

In real world, we may design complex inheritence diagrams and just create one instance of them, in this case **modules** may be a better solution.








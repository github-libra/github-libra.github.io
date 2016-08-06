---
title: ES6 Fundamentals
category: javascript
tags: es6, javascript
---

## Variables & Parameters

### let keyword

`let` creates block scope. It avoids a lot of confusion and mistakes with javascript declaration with `var`. Like

```javascript
for(var i=0; i<10; ++i) {

}
i; // 10

for(let i=0; i<10; ++i) {

}
i; // not defined error
```

### const keyword

`const` creates an unmutable variable with block scope.

```javascript
const a = 1;
a = 2; // TypeError, Assignment to constant variable;
```

### Destructuring

Pick up some elements or properties of an array of an object.

```javascript
let [, x, y, z, a] = [1, 2, 3, 4];
x; // 2
y; // 3
a; // undefined

let { first: firstName, last: lastName } = { first: 'HJ', last: 'Tang' };
firstName; // 'HJ'
lastName; // 'Tang'

// or if keep the key
let { first, last } = { first: 'HJ', last: 'Tang' };
first; // 'HJ';
last; // 'Tang'
```

Destructuring can be handy when dealing with function parameters. eg:

```javascript
function http(url, {data, method, headers}) {
    return data;
}

var result = http('api/users',
    {
        data: 'thj',
        method: 'GET'
    });
expect(result).toBe('thj');
```

### Default Parameter

```javascript
// often in code
function add(a, b, c) {
    a = a || 1;
    b = b || 2;
    c = c || 3;
    return a + b + c;
}

// default parameter
function add(a=1, b=2, c=3) {
    return a + b + c;
}

// or together with destructure
function http(url, {data='thj', method='GET'}) {
    return method;
}
let result = http('api/users', {data: 'someone'});
expect(result).toBe('GET');
```

### Rest parameters

```javascript
// often in code
function sum() {
    var args = [].slice.call(arguments),
        sum = 0;  
    args.forEach(function(each) {
        sum += each;
    });
    return sum;
}

function sum(...numbers) {
    let sum = 0;
    numbers.forEach(function(each) {
        sum += each;
    });
    return sum;
}
```

`...` operator is greedy, it should be the last parameter of functions, it tries to take all of the *rest* parameters into it. 
If no arguments passed, `...` defaults to `[]`;
Compares to implicit `arguments`, `...` resolves an actual Array;

### Spread Operator

Just like *Rest parameters*, when `...` is used before an array, it's called *spread operator*, it saparates array elements into arguments list.

```javascript
function do(x, y, z) {
    return x + y + z;
}
do(...[1, 2, 3]);

// or
let a = [4, 5, 6];
let b = [1, 2, 3, ...a, 7, 8, 9];
```

### Templates Literals

help build strings instead of string concat.

```javascript
let name = 'thj';
let greeting = `Hello, ${name}`;
```

## class


## Functional Programming














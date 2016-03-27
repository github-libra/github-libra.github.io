---
title: How angular works -- Creating-Objects
tag: angular, provider
category: angular
---

<p class="lead">
    Down to the earth, they're just $provider.
</p>

We use `angular.module()` to declare module, but how do we create components in a module?

We're familiar with creating components like this:

```javascript
// declare components in app module
angular.module('app')
    .constant('SERVER', 'http://url-to-server')
    .value('DATE_FORMAT', 'YY-MM-DD')
    .controller('AppCtrl', ['$scope', function() { }])
    .service('someService', [function() { }])
    .factory('someService', [function() { }])
```
Well, down to the earth, they're `$provider`


### what's $provider?

>The $provide service has a number of methods for registering components with the $injector. Many of these functions are also exposed on angular.Module.

so $provider has several ways to create components:

- provider(name, provider)
- factory()
- service()
- value()
- constant()
- decorator()

`factory`,`service`,`value`,`constant` are just syntax sugar of `provider`, they call provider internally.

The second argument of `provider()` can be an Object or a function. If it's an Object, it should have a $get method which will be call by `$injector.invoke()`. If it's a constructor, object will be created by `$injector.instantiate()`.

A provider example would be:

```javascript
$provider.provider('log', function(){
    var mode = 'debug';

    this.setMode = function(mode) {
        this.mode = mode;
    }
    this.$get = function() {
        if(this.mode === 'debug') {
            return console.debug;
        }
    }
})
```


### how to use $provider?







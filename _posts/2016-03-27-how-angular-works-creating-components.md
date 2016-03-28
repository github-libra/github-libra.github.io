---
title: How angular works -- Creating-Components
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


## What's $provider?

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



## How to use $provider?

Normally syntax sugars are enough, but if we want to create a reusable components that can be configured before running, `provider` can do this.

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
        }else if(this.mode === 'trace') {
            return console.trace
        }
    }
})

// config before running

angular.module('app')
.config(['logProvider', function(logProvider) {
    logProvider.setMode = 'trace';
}])
.controller('Ctrl', ['log', function(log) {
    log('will log trace');
}])

```

## How about the sugar syntax?

`constant` and `value` are useful to define application wide constants. The only difference is `constant` is available both config and run phase, while `value` can only be used in run phase. So we can use `constant` to define some configuration constant used in provider during config phase.

`factory` can has dependency and logic, it will be lazy load when needed. We need to return something inside the function. 

`service` can be useful when we write JS classes. For example:

```javascript
function Rocket(code) {
    this.country = "Russia";
    this.launch = function() {
        if(code === 'MANBA') {
            alert('launch!');
        }
    }
}

angular.module('app')
.service('rocketService', Rocket);

```

## Special purpose objects

>we also have special purpose objects that are different from services. These objects extend the framework as plugins and therefore must implement interfaces specified by Angular. These interfaces are Controller, Directive, Filter and Animation.

The instructions for the injector to create these special objects (with the exception of the Controller objects) use the Factory recipe behind the scenes.

So `Filter`, `Directive`, `Animation` all return objects or functions but `Controller` doesn't. And `Controller` can use it's dependencies as it **constructor** arguments.

```javascript
myApp.controller('DemoController', ['clientId', function DemoController(clientId) {
  this.clientId = clientId;
}]);
```

## Wrap it up

From Angular [docs](https://docs.angularjs.org/guide/providers)

>* The injector uses recipes to create two types of objects: services and special purpose objects
* There are five recipe types that define how to create objects: Value, Factory, Service, Provider and Constant.
* Factory and Service are the most commonly used recipes. The only difference between them is that the Service recipe works better for objects of a custom type, while the Factory can produce JavaScript primitives and functions.
* The Provider recipe is the core recipe type and all the other ones are just syntactic sugar on it.
* Provider is the most complex recipe type. You don't need it unless you are building a reusable piece of code that needs global configuration.
* All special purpose objects except for the Controller are defined via Factory recipes.

|Features Recipe type |  Factory | Service | Value | Constant | Provider |
|---|---|---|---|---|---|
|can have dependencies |  yes | yes | no | no | yes |
|uses type friendly injection  |  no | yes | yes*  | yes* | no |
|object available in config phase |   no | no | no | yes | yes** |
|can create functions | yes | yes | yes | yes | yes |
can create primitives  | yes | no | yes | yes | yes |
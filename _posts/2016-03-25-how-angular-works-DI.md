---
title: How angular works -- Dependency Injection
tag: angular, injector
category: angular
---

<p class="lead">
    Angular offers Dependency Injection(DI) which simplifies the way we manage components and tests.
</p>

## why do we need that?

As documented @[AngularJS developer guide]()

There are only three ways a component (object or function) can get a hold of its dependencies:

>1. The component can create the dependency, typically using the new operator.
2. The component can look up the dependency, by referring to a global variable.
3. The component can have the dependency passed to it where it is needed.

Clearly, we *hardcode* the dependency either by `new` operator or maintain a dependency map. Using the 3rd method, we can easily get what we want without the need to create one.

## what is injector?

Under the hood, Angular **Injector** does all the work locating the `services`, instantiate `controllers`. It manages the dependencies so that other services just ask what they want.
From the document:

>  `$injector` is used to retrieve object instances as defined by $provide, instantiate types, invoke methods, and load modules.

## how does injector work?

`Injector` has 5 methods attached to it. They are

- get(name), get the instance we need
- has(name), return true if injector has the instance
- invoke(fn, self), call the specified function if it doesn't have one
- instantiate(Type), create `controller`
- annotate(fn), return dependency names

`invoke()` and `instantiate()` are main functions.

As we know, `services` in Angular are singleton. So the first time our service B ask for  service A, `injector` will create an A service using `invoke`(for factory syntax) or `instantiate`(for service syntax) internally, later we want A again, `injector` just returns the A it already has.

However, `controllers` in Angular are not singleton. `Injector` calls `instantiate` to create a `controller` every time the page need one.

For `annotate()`, Angular has 3 ways to specify dependencies.

### By argument

```javascript
function AController($scope, $route) {
    // controller logic here
}
```
Note that, annotate by argument is not safe with minification, it may changes the argument names to a or b, then `injector` won't know the dependencies.

### By $inject property

```javascript
AController.$inject = ['$scope', '$route'];
function AController(scopeService, routeService) {
    // controller logic here
}
```

### By array syntax

```javascript
['$scope', '$route', function AController(scopeService, routeService) {
    // controller logic here
}];
```











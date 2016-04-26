---
title: How angular works -- Directive
tag: angular, directive
category: angular
---

<p class="lead">
    Create something new!
</p>

## What are Directives?

>At a high level, directives are markers on a DOM element (such as an attribute, element name, comment or CSS class) that tell AngularJS's HTML compiler ($compile) to attach a specified behavior to that DOM element (e.g. via event listeners), or even to transform the DOM element and its children.

## how to define a directive?
 
Angular uses the factory recipe to define directive, the API of directive is:

```javascript
angular.module('app')
.directive('menu', ['$window', function($window) {
    return {
        priority: 0,
        restrict: 'EACM',
        require: '^ngModel', //require other directives, ^ means looks in parents
        template: '',
        templateUrl: '', // if not template
        scope: { // create isolated scope
            localName: '@domAttr' // one-way
            information: '=?info', // two-way
            onClick: '&' // delegate function
        },
        compile: function(tElement, tAttrs, transclude) {
            return {
                preLink: function() {},
                postLink: function() {}
            }
        }, 
        link: function(scope, element, attrs, controller, transcludeFn) {},
        transclude: true, // wrap other elements, work with ngTransclude directive
        controller: function($scope, $element, $attrs, $transclude, otherInjectables) {},
        bindToController: true,

    }
}])

```

### scope

The scope property can be true, an object or a falsy value:

* falsy: No scope will be created for the directive. The directive will use its parent's scope.

* true: A new child scope that prototypically inherits from its parent will be created for the directive's element. If multiple directives on the same element request a new scope, only one new scope is created. The new scope rule does not apply for the root of the template since the root of the template always gets a new scope.

* {...} (an object hash): A new "isolate" scope is created for the directive's element. The 'isolate' scope differs from normal scope in that it does not prototypically inherit from its parent scope. This is useful when creating reusable components, which should not accidentally read or modify data in the parent scope.

### require

`require` is a property that allow directive to communicate with other directive. Use it by specify the directive name if they're on one element(or array of names). Use the `^` prefix to specify looking ups its parent. With `require` the controllers will be available to the `link()` at the 4th parameter.

>(no prefix) - Locate the required controller on the current element. Throw an error if not found.
? - Attempt to locate the required controller or pass null to the link fn if not found.
^ - Locate the required controller by searching the element and its parents. Throw an error if not found.
^^ - Locate the required controller by searching the element's parents. Throw an error if not found.
?^ - Attempt to locate the required controller by searching the element and its parents or pass null to the link fn if not found.
?^^ - Attempt to locate the required controller by searching the element's parents, or pass null to the link fn if not found.

### bindToController

if it's true, the variables bind to scope will bind to controller instead. And it accepts object hash. If both `scope` and `bindToController` are set, `bindToController` overrides `scope`.
















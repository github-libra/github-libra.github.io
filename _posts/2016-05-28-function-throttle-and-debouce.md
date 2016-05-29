---
title: Function De-bounce and Throttle  
tag: javascript, performance
category: javascript
---

## Problem

Often I write JS code like this:

```javascript
window.addEventListener('mousemove', onMouseMove);
function onMouseMove(e) {
    // listener function
}
```

Events such as `mousemove` and `resize` will trigger many times(~56) in 1 second. If I write complex code in listener that takes relatively long time to run, JS thread will be very busy just executing my listener function instead of responding to user interaction.


## Solution...

Here's comes the concept of **function debounce**. I want the listener only be called after some time I stop *resizing* window or *moving* the mouse. I could write a debounce utility:

```javascript
function debounce(listener, delay) {
    var timeId;
    delay = delay || 1000;
    return function() {
        var args = arguments,
            self = this;
        clearTimeout(timeId);
        timeId = setTimeout(function() {
            listener.apply(self, args);
        }, delay);
    }
}
// usage
window.addEventListener('mousemove', debounce(onMouseMove));
function onMouseMove(e) {
    // listener function
}
```

The `debounce` function wraps `listener` function within a timeout. Every time the returned function get called, it cancels previous timeout and set a new one. Since it's a wrapper, it should maintain the same executing context and argument list as normal listener function call.

## Problem again...

I might find a perfect solution for `mousemove` and `resize`, but what if it's a `drag` event listener, the `debounce` utility will result in listener function called only after some time I stop dragging. This is not often what I want for dragging cause it will confuse user.

So this time I need listener get called not only during dragging but not so often. I can pass another argument to set the timespan to execute listener.


## Solution

```javascript
function throttle(listener, delay, must) {
  var timeId,
      startAt = +new Date();
  delay = delay || 100;
  must = must || 300;
  return function() {
      var args = arguments,
      self = this,
      now = +new Date();
      clearTimeout(timeId);
      if (now - startAt > must) {
          listener.apply(self, args);
          startAt = now;
      } else {
          timeId = setTimeout(function() {
              listener.apply(self, args);
              startAt = +new Date();
          }, delay);
      }
  }
}
```
In the throttle function we calculate the timespan between each listener executing. If it's longer than the must execute timespan, call the listener function and reset the `startAt` time. In this way listener function at least be called every 300 milliseconds.

## Is this in fact efficient?

These 2 solutions seems to be able to deal with my problem, but down to earth, does they outperform the native way in the point of real world browser environment?

To know this, I use Chrome profile to capture CPU usage. I write a nested for loop in the listener which takes about 200 ms to execute. Here's the captured result of CPU and memory usage using *native/debounce/throttle*.

![js-function-throttle-profile](/images/js-function-throttle-profile.png)

From the result it's easy to see the latter two ways to add listener to frequently called event can save computing resources and memory.







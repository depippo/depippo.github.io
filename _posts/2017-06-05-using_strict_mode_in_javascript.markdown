---
layout: post
title:  Using 'strict' mode in Javascript
date:   2017-06-05 00:07:09 +0000
---


There are a number of benefits to using 'strict' mode in Javascript. In general, these fall under the umbrella of better and more predictable error handling. Strict mode alerts us to potential problems in our code that might otherwise have gone unnoticed. Let's take a look at some of the most significant examples of how strict mode differs from default Javascript.

#### Global Variables

Outside of strict mode, Javascript will allow us to set a variable without first declaring it. For example, the following will not throw any errors in standard Javascript:

```
var myFunction = function() {
  x = 2;
}
```

This is technically valid, but by setting the variable x without declaring it, we've actually created a global variable. We've set x to equal 2 globally, not just in the scope of the function we were defining. Most of the time, it's unlikely that this is what we intended, as this could lead to some unpredictable behavior. If we were to try to write this same function in strict mode however, we'd get an error message:
> ReferenceError: x is not defined

In strict mode, we have to make sure we declare the variable, like so:

```
'use strict';
var myFunction = function() {
  var x = 2;
}
```

Now x is defined, and is available as a local variable within myFunction as we most likely intended.

(Note: if we really wanted to set a global variable from within our function in strict mode, we could still do it by explicitly stating window.x = 2.)

#### This coercion

By default, Javascript will allow us to reference "this" from within a function that is not bound to an object. For example, when we run the following code:

```
var myFunction = function() {
    console.log(this);
}
myFunction();
```

we'll see the following printed out in the console, because we're referencing the window object which contains the function:

> Window {stop: function, open: function, alert: function, confirm: function, prompt: function…}

Similar to the way Javascript implicity sets global variables like we just saw above, it will by default set "this" to the global context.

If we were to run this same function in strict mode however, "this" would be undefined. We could still employ the very same function witihin strict mode if we subsequently use call, apply, or bind with the functon, and specify a "this" for the function. In the following example, when we call our function, we're specifying that "this" will be the "name" object:

```
'use strict';
var name = "David"
var myFunction = function() {
    console.log(this);
}

myFunction.call(name);
// "David"
```

Within strict mode, "this" can also still be used to refer to the object context in which the function is contained. In the following example, "this" will refer to the "person" object:

```
'use strict';
var person = {
    name: "Frank",
    sayName: function () {
        console.log(this.name);
    }
};

person.sayName();
// "Frank"
```

#### Read-only properties

In Javascript, there are certain global properties which are, for pretty obvious reasons, read-only:

* Infinity
* NaN
* undefined

Let's say we attempted to assign a value to one of these properties, i.e.:

`var undefined = "defined";`

This code would "fail silently." That is, it won't actually alter the value of undefined, but neither will it throw an error message. The above line of code is almost certainly a mistake though, so running in strict mode could help us out here.

If we had enabled strict mode and attempted to modify the value of undefined, we'd get the following error message:

> Uncaught TypeError: Cannot assign to read only property 'undefined' of object '#<Window>'

This is much better, because now we know that whatever we were attempting to do by assigning a value to undefined, it did not work, and we need to revisit that line of code.

These are some of the more salient differences between strict mode and default Javascript, but there are several more. For further reading, check out the following docs:

[Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) - Mozilla Developer Network

[JavaScript Use Strict](https://www.w3schools.com/js/js_strict.asp) - W3 Schools

[Strict Mode (JavaScript)](https://docs.microsoft.com/en-us/scripting/javascript/advanced/strict-mode-javascript) - Microsoft Docs





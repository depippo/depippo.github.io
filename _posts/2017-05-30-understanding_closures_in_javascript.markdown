---
layout: post
title:  Understanding closures in Javascript
date:   2017-05-30 11:18:11 -0400
---


In my last blog post, I used an interview question I’d recently been asked as a jumping off point to talk about data-binding. I’d like to do something similar this week, this time taking my cue from a question about Javascript closures. It’s another concept that is important to understand for Javascript developers, and is also likely to come up in technical interviews. So, what is a closure in Javascript?

The Mozilla Developer Network docs offer the following definition:

> Closures are functions that refer to independent (free) variables (variables that are used locally, but defined in an enclosing scope). In other words, these functions 'remember' the environment in which they were created.

I think the best way to understand what that means and why it matters is to start by looking at some examples. Here is a very simple function that just console logs a basic message.
```
function sayHello() {
  var name = "Buster";
  var greeting = function() {
    console.log("Hello " + name);
  }
  greeting();
}
```

If we then call:

`sayHello();`

We'll get 'Hello Buster' printed out to the console. 

Nothing fancy, but this is a closure; the inner function 'greeting' has access to the name variable, which is defined in its enclosing scope. Anytime we call the sayHello function, the name variable will be successfully accessed to form the greeting. However, if we were to try to access the name variable outside of this function, we'd get the error message 'Uncaught ReferenceError: name is not defined.' If we had defined the name variable outside the sayHello function, i.e., in the global scope, we would naturally be able to access it within the function, but the variable would also be accessible to other parts of the application unnecessarily, which might lead to some behavior we don't want.

We can make this a bit more useful by updating the function to accept a name parameter:

```
function sayHello(name) {
  var script = 'Hello ' + name;
  var greeting = function() {
    console.log(script);
  }
  greeting();
}
```

Now we can call sayHello with whatever name we want. This demonstrates that closures can not only read variables in their enclosing scope, but modify them as well.

`sayHello("Nigel"); // logs 'Hello Nigel'`

`sayHello("Lily"); // logs 'Hello Lily'`


Let's take a look at a slightly more complicated and useful example of a closure.

```
function add() {
  var counter = 0;
	
  return {
    increment: function() {
      counter++;
    },
    print: function() {
      console.log(counter);
    }
  }
}
var a = add();
```

Here again, we have a variable that cannot be accessed outside of the enclosing function, as we don't want any other part of the application to be able to change the counter. We also have a couple of inner functions, increment and print, that can access the counter variable, and which we can subsequently call like so:

```
a.print(); // logs 0
a.increment();
a.increment();
a.print(); // logs 2
```

We can't just call a.counter; the only way to access the value of counter is via the "privileged" functions we defined within add(). Creating a new variable equal to the add function gives us easy access to two functions (increment and print) that can be used anywhere in our application to manipulate or return the "private" data of the add function in a predictable and controlled fashion, without having to expose the data to the global scope.

Check out the following links for some great further reading on the subject of closures:


[JavaScript Closures](https://www.w3schools.com/js/js_function_closures.asp) - W3 Schools

[Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) - Mozilla Developer Network

[Private Members in JavaScript](http://javascript.crockford.com/private.html) - Douglas Crockford





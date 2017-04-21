---
title: 'behind the scenes of JavaScript functions'
date: '2017-04-19 22:12:00'
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

This post is about what happens behind the scenes when you define a new function in JavaScript; and the multiple ways the said function can be used.

Take the following example.  A simple function called *Util* that has another function inside of it. 

```javascript
function Util() {
  
		 console.log('from Util');	
  
		 this.printer1 = function (param) {
             console.log(param);
  	     };
}
```

In JS, all variables and functions are stored inside objects, and those in turn are stored inside more objects; until finally there is a single object that holds everything. This object is your usually the browser window.

In JS, functions can be stored in and passed around like variables. Also, remember that functions (both constructors and instances) are objects.



### Objects in JS

Now objects in JavaScript consist of key-value pairs. To see what's in your window object, open up a browser window, or a [new fiddle](https://jsfiddle.net/6tdp5gaw/), and enter `console.dir(window)` in the console. 

![console](http://uvinw.github.io/assets/images/consolewindow.png)

Everything in your page (variables, arrays, objects, functions) reside in this window object as key-value pairs. So what does a function declaration look like? Paste the entire *Util* function definition from above in the console and rerun `console.dir(window)`. Now `Ctrl+F 'Util'` and you'll have your answer.

The scope will now contain a new key called *Util* and it's value will be the function we defined above.

![util function in console](http://uvinw.github.io/assets/images/consolewindow.png)

To be more specific here, what we did above was actually define a *function constructor*.  If you check the current window scope, there will be a *Util* function, but no *printer1* function. 

Now run the constructor like this `Util(); ` and then rerun `console.dir(window)` to take a peek at the scope object. Now the function constructor has executed. This process assigns the function constructor *printer1* to a new key in the execution context. Search for *printer1* and a new object can be found. 

So in essence, what the line `this.printer1 = function... ` does, is that is creates a new key-value pair in the current context, which in this instance is the window scope. So the scope object now has a new function which can be called.

```javascript
printer1('sup');
> sup
```

Remember, most things are objects in JS. Sort of.



### New Keyword

```javascript
var myUtil = new Util();
```

When using the new keyword on a function, a sequence of events take place. 

A new variable called *myUtil* is defined and assigned a new object. This object automatically inherits the prototype object (cause its JavaScript). This new object is also passed the function constructor called *Util*. Note that this runs the function constructor (*Util*), but in a different scope. This scope is inside the myUtil variable (or rather its value as 'myUtil' is simply a key in the parent scope).

The function *printer1* will not be available as a function in the window scope, but will be inside the *myUtil* scope. Because every time a new object is created in JS, a new scope is created. And right now the execution sequence is inside the myUtil scope and that's where the new function object (of *printer1*) will be created. 

So in summary, no *printer1* function is found in the window scope, but is found in the *myUtil* scope. 

```javascript
printer1('sup');
> Reference Error

myUtil.printer1('sup');
> sup
```



### Post Definition

Going back to the code, let's add a new function to *Util*, after it is defined

```javascript
function Util() {
  	console.log('from Util');	
  	this.printer1 = function (param) {
         console.log(param);
  	};
}

Util.printer2 = function (param) {
    console.log(param);
};
```

Here's what happens behind the scenes now. 

Util, the function constructor object is accessed from the current scope and a new key-value pair is added to the object. The key being *printer2* and the value being a new function constructor. The scope where this new object is added is inside the existing Util constructor. Why? Because we called `Util.`.

Print the window scope again using `console.dir(window)` and look for the *Util* key. This key will now contain both the function constructor for our *Util* function (this is not visible in the console as this is an object constructor, but know that it's there), as well as a another key called *printer2* which holds a new function constructor for the *printer2* function. 

```javascript
Util : {
  printer2 : function (param) {
    console.log(param);
  },
  __proto__ : ...
}, 
foo :  {
  //another object in the window scope
},
```

This means that *Util* is a key found inside the window scope that holds a reference to its function constructor. This function does not exist in the current scope, only the key (a reference to the constructor) does. And the constructor exists **inside** the object (value of the said *Util* key). The new function *printer2* is now in this scope, and in the same hierarchy as the *Util* constructor (inside the object that is pointed to by the *Util* key).

So without running any function constructors (and without having to create a new Util function object), we can directly call *printer2* like so.

```javascript
Util.printer2('yo');
> yo
```

So a new object of Util is not made, but *printer2* is accessible from within that object (the function constructor) and can be called. 

I know this is hard visualize, but it happens to be how JS works. Whew.
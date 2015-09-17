---
layout: post
title:  "this is Javascript!!"
last_modified_at: 2015-03-15T05:20:00+05:45 # last page modified date/time
date:   2015-09-15 12:19:35
categories: Javascript this
imageTitle: This is Spa...JAVASCRIPT
comments: true
featured_image: this-is-javascript.jpg
---

Nope, I am not Leonidas nor this is Sparta. In this post I am going to shed some light on the 
mystical and magic **this** object. I have been writing Javascript for almost a decade now, and one 
of the (many) things that always was a pain was that *self* reference that never seemed to be what I 
was expecting it to be.  
So let's get rid of all the mystery and magic around it and try to make amends 
with **this**.

## What is *this*?

There are many misconceptions out there about what *this* means. A lot of people tend to believe that it is a 
reference to 'itself', and that is only half true. It can be the same as 'itself' sometimes but it can be not, so not a valid theory.  
The other theory I've heard is that *this* represents the *scope* of the function, but it is also wrong. 
First, we need to understand that there is no way to access the *scope* trough Javascript, because it is mainly a part
of the Javascript Engine that gets created at compiling time. So no, sorry it is not the scope neither.

Ok buddy, so what is 'this'?  
*this* is a special keyword that every single function have in Javascript. It is automatically defined based 
on a sort of rules and it is pretty useful, actually, because it allows you to control the *context* (remember *context !== scope*) 
a function runs against. Why would we do that? To make our code cleaner, and reusable. That does not sound too
bad, does it?  

Let's see an example to show why **this** has so many haters out there.

{% highlight js %}

var variable = 'Global';

function func(){
  console.log(this.variable);
}

var obj = {
  variable: 'Inside Obj',
  function: func
};

var anotherFunc = obj.function;

func(); //'Global'

obj.function(); //'Inside Obj'

anotherFunc(); //'Global'

new anotherFunc() //undefined

{% endhighlight %}

Maaaan, that is way too much! The same function called in four different ways and every one of them (almost) have a
different **this** object. Sometimes it is the global object, sometimes it is not and sometimes it is even undefined! How
does this stuff work?


## Choosing *this*

Ok, we just saw how **this** can change its content, now we are going to understand how this content is decided. 
It does not depend on the function definition as you could imagine from the previous piece of code, it is decided by
they way the function is called. Sometimes it can be tricky to know exactly how the function is called but no worries, 
let's see the four ways a function can be called and how that affects the **this** object.

### Default binding

Default binding happens when none of the other binding methods apply. It's behavior depends on
the mode where the function is **called**. If we are in *strict mode* **this** will be equal
to null, to avoid leaking any variables into the global scope. Otherwise, it defaults to 
*window*, and we need to be extremely careful with this. 

{% highlight js %}
var variable = 'Global';

function defaultFunc(param){
  console.log(this.variable);
  this.variable = param;
}

function strictFunc(param){
  'use strict;'
  defaultFunc(param);
}

defaultFunc('Modified'); //'Global'

console.log(variable); //'Modified'

strictFunc('Strict'); // Throws a null reference error, because this === null

{% endhighlight %}

As a general rule of thumb, always try to use *strict mode* and if you can't, 
avoid mixing modes as much as possible.


### Implicit Binding

This kind of binding applies when Javascript is able to choose a better context than
the global based on the call-site.

{% highlight js %}
var variable = 'Global';

var obj = {
  variable: 'Object',
  callMe: function(){
    console.log(this.variable);
  }
};

obj.callMe(); // 'Object'

{% endhighlight %}

It is important to notice that only the function's parent object is relevant.

{% highlight js %}
var variable = 'Global';

function func(){
  console.log(this.variable);
}

var obj = {
  variable: 'Object',
  func: func,
  innerObject: {
    variable: 'Inner',
    func: func
  }
};

obj.func(); // 'Object'

obj.innerObject.func(); // 'Inner'
{% endhighlight %}

Sweet! An easy one! So, every time a function is inside an object **this** will 
be the object, right? **NOPE**, not right. Remember that the important thing 
choosing **this** is the **call-site**.

{% highlight js %}
var variable = 'Global';

var obj = {
  variable: 'Object',
  callMe: function(){
    console.log(this.variable);
  }
};

var reference = obj.callMe;

reference(); // 'Global'

{% endhighlight %}

Be careful with these things! Here we are copying a reference to the function 
itself inside our *reference* variable, when we call it we don't have any reference to 
*obj* so the Default Binding is applied and **this** is the global scope.

### Explicit Binding

Sometimes it is useful to choose the value of **this** for a function and, of course, 
there is a way of doing this in Javascript, three of them actually, **call**, **apply** and **bind**.

{% highlight js %}
var variable = 'Global';

var obj = {
  variable: 'Object',
  callMe: function(){
    console.log(this.variable);
  }
};

var newObj = {
  variable: 'New'
};

var reference = obj.callMe;

reference.call(obj); // 'Object'
reference.apply(newObj); // 'New'

obj.callMe.call(newObj); //'New'
{% endhighlight %}

For binding purposes, **call** and **apply** are interchangeable, they have some 
differences when it comes to dealing with parameters.  
* **call** expects the parameters one after the other, in the same order as the original
function.
* **apply** takes an array as second parameter and use those values in order as the
parameters of the original function.

Which one to use is totally up to you.

**bind** is a little bit different, it creates a new function that calls the original function
with the specified context, independently of its own context. Confusing right? Not that much, let's see.

{% highlight js %}
var variable = 'Global';

var obj = {
  variable: 'Object',
  callMe: function(){
    console.log(this.variable);
  }
};

var newObj = {
  variable: 'New'
};

var reference = obj.callMe;

var bindedReference = reference.bind(newObj);

var bindedImplicit = obj.callMe.bind(window);

bindedReference(); //New

bindedReference.call(obj); //New

bindedImplicit(); //'Global'

bindedImplicit.apply(obj); //'Global'
{% endhighlight %}

There we go, you cannot modify *this* in a function that has been previously created
by **bind**. Well, you should never say never...

### new Binding

Here we go! The **new** keyword, one of the main causes why people use
Javascript in the wrong way! This little keyword has nothing to do with classes
or Object Oriented Programming in the classical way. It is just one of many ways that Javascript has to specify the value of **this** (and some other things that are out
of the scope of this post)
The **new** keyword works kind of like this.

1. It creates a new Object.
2. It assigns that new Object to **this**
3. It makes the function to return that Object.

{% highlight js %}
var variable = 'Global';

var globalFunction = function(){
  console.log(this.variable);
};

var obj = {
  variable: 'Object',
  callMe: globalFunction
};

new globalFunction(); //undefined
new obj.callMe(); //undefined
{% endhighlight %}

This examples all return *undefined*, and that is because their **this** are
this newly created Object we were talking about!

This is cool, but we need to be extremely cautious because using **new** in the wrong place can make things fail (and it is a tricky bug to find), but not using the **new** keyword when you are supposed to... aaaah buddy, that is REAAAALLY BAD.

{% highlight js %}
function Person(name, age){
  this.name = name;
  this.age = age;
  
  this.speak = function(){
    console.log('Hi there, I am ' + this.name);
  }
}

console.log(name); //"ReferenceError: name is not defined"
console.log(age);  //"ReferenceError: age is not defined"

var javi = new Person ('Javi', 30);

javi.speak(); //"Hi there, I am Javi"

console.log(name); //"ReferenceError: name is not defined"
console.log(age);  //"ReferenceError: age is not defined"

var ragnar = Person('Ragnar', 1000);

ragnar.speak(); //"TypeError: Cannot read property 'speak' of undefined"

console.log(name); //"Ragnar"
console.log(age); //1000
{% endhighlight %}

WOW! Look at that mess, we have leaked variables everywhere. And think what can happen if any of those variables previously existed and some library were relying on them...  
So, how can we prevent this? Sticking to conventions, a function that starts with a Capital letter should always be used with **new**

## Wrapping things up

No excuses from now on! Everything should be crystal clear when it comes to dealing with **this**.  
Just as a quick recap for finding out what is this:

1. Is the function called with **new**? **this** is a newly created object.
2. Is **this** explicitly bind with **call**, **bind** or **apply**? **this** would be what we passed to those functions.
3. Is the function inside an object **in the call-site**? **this** would be that object.
4. If you are here, then **this** is the global object **window**

I hope you have enjoyed this reading, if you are interested in getting even in deeper detail about what is going on under the hood of Javascript I can't do more than
recommending this books [You don't know JS], they are AMAZING! Go get'em!

See you around!

[You don't know JS]: https://github.com/getify/You-Dont-Know-JS

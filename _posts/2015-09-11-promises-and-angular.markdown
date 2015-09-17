---
layout: post
title:  "Promises & Angular"
date:   2015-09-11 12:19:35
categories: AngularJS, promises
imageTitle: Do you trust me? Then...
featured_image: http://www.javierporrero.com/blog/assets/promises.jpg
---

Promises are one of the best things ever happened to Javascript, but as good as they are, they can be hard to get in the right way. This is why I created this post, I see a lot of people out there using promises, but some of them are not using them properly or not using all the power that this little friends can give us. So, let's get started with some general stuff.


Definition & Terminology
-------------------------

Let's get this straight, there is an official way to call things as per the [official specification], so we will stick to it.

First thing we need to know, what a Javascript Promise is.

>A Promise is an object that is used as a placeholder for the eventual results of a deferred (and possibly asynchronous) computation.

Ok, roger that. So we can say that a Promise is a wrapper around an action(asynchronous or not) that waits for the outcome of that action. Sounds nice.

A Promise can go through some different states.

* **Pending** - The action is not finished yet
* **Fulfilled** - The action we were waiting for is successfully finished
* **Rejected** - The associated action failed
* **Settled** - The promise is fulfilled or rejected

It is important to know a few key things about promises:

* A settled promise can't change it's state, it is rejected or fulfilled.
* A promise can only succeed or fail **once**.
* A callback attached to a settled promise will always be called based on the state of the promise. This is, if you add a sucess callback to a fulfilled promise it will be called straight away.


Promises in Javascript
-----------------------------

Ok, let's get our hands dirty with some code. Promises in Javascript are meant to be used like this.

{% highlight js %}
var promise = new Promise(function(resolve, reject){
  var result = doSomeStuff();
  if(result === 'Everything went well'){
    resolve('Yay!');
  }else{
    reject('It failed!');
  }
});

promise.then(function onSuccess(result){
  console.log('result'); // 'Yay!'
}, function onError(error){
  console.log(error); //'It failed!'
});
{% endhighlight %}

Promise constructor takes just one argument, a function with two parameters. We can do whatever we want inside this function, 
if it worked we call the first parameter (*resolve*), in case it didn't work we call the second parameter (*reject*).

There is another way to reject the promise, and that is Exceptions. If we throw and error or leak one inside the constructor's callback, 
the promise will be automatically rejected, taking the error thrown as a parameter.

Every promise object has a method **then**, this method takes two optional arguments. The first one is a callback that will be called 
if the promise resolves, the second one is another callback for when the promise rejects. Both callbacks get as a parameter
whatever we passed to *resolve* or *reject* as parameter in the promise function. As both are optional you don't need to always define them.

Maybe an example works better than words.

Here we wrapped an AJAX call to *'/number'*, an endpoint that returns a random integer in the range [0...100]. Then, we call 
resolve or reject depending on the outcome of the request, passing the response. In case there is any kind of error making 
the request we reject passing a String. 
{% highlight js %}
function getRandomNumber(){
  return new Promise(function(resolve, reject){
    var request = new XMLHttpRequest();
    request.open('GET', '/number');

    request.onload = function(){
      //we resolve or reject depending on the status code
      //of the request always passing the response as a paramater
      ((request.status === 200)?resolve:reject)(request.response);
    }

    request.onerror = function(){
      //In case there was an error we also reject
      reject('Error');
    }

    request.send();
  });
} 
{% endhighlight %}

Ok, now it is time to use this new function and get a random number.
{% highlight js %}
getRandomNumber().then(function(number){
    console.log(number); //Our random number!
  }, function(error){
    console.log(error); //Oops
  });
{% endhighlight %}

Chaining promises
==================

Easy huh? There is much more than just this! What if I tell you that you can **chain** *then* functions? Every then function pass
whatever their callback returns to the next *then* function. Even better, if the returning value is a **promise** it will wait for it
to settle before calling the next *then* function. Awesome right? Let's try this out.

{% highlight js %}
getRandomNumber().then(function(number){
    //As we are returning a non-promise object,
    //the next .then will be executed immediately
    return number*2; 
  }, function(error){
    console.log(error); //Oops
  }).then(function(numberDoubled){
    console.log(numberDoubled); //Yup, here we have the number doubled!
  });
{% endhighlight %}

Now let's chain some promises.
{% highlight js %}
getRandomNumber().then(function(number){
    console.log(number); //Prints the first number
    //We are returning a promise object here,
    //the next .then will wait for it
    return getRandomNumber(); 
  }, function(error){
    console.log(error); //Error in the first promise
  }).then(function(secondNumber){
    console.log(secondNumber); //Prints the second number
  }, function(error){
    console.log(error); //Error in the second promise
  });
{% endhighlight %}

Sweet. But what if we want to request 20 random numbers? Doing it this way could be a nightmare, but here is when a new Promise 
method comes to the rescue: **all**.

Our new friend **all** takes an array of promises as argument and returns a new Promise that will resolve when *every* promise in the array is fulfilled,
passing an array with the results, in the same order as requested, to *then*.

{% highlight js %}
var promises = [];
for(var i=0; i<20; i++){
  //populates the array with some promises
  promises.push(getRandomNumber());
}

Promise.all(promises).then(function(responses){
  console.log(responses); //Array of responses of every request
}, function(err){
  console.log(err); //Error of the first promise rejected
});

{% endhighlight %}

Error handling
==============

Promise objects always send errors to the next *then* method until they find an error callback, so we can write our code like this

{% highlight js %}
Promise.all(promises).then(function(responses){
  console.log(responses); //Array of responses of every request
}).catch(function(err){
  console.log(err); //Error of the first promise rejected
});

{% endhighlight %}

In this piece of code, **catch** is just a helper, it is actually the same as writing *.then(null, function(){})*. But it looks cool
and is very readable!

Additional (interesting) stuff
==============================

There is still some method in the Promise object that are worth knowing.

{% highlight js %}
Promise.resolve() // returns a fulfilled Promise
  .then(function(){
    console.log('Yay!');
  }, function(){
    console.log("Doesn't matter"); //This callback will never be called
  });

Promise.reject() // returns a rejected Promise
  .then(function(){
    console.log("Doesn't matter"); //This callback will never be called
  }, function(){
    console.log('Yay?');
  });
{% endhighlight %}

Oooooook, that was quite an introduction to the subject... Now it is time to see how we can use all this stuff in Angular.

Promises in Angular
-------------------

If Promises are a standard thing in Javascript, how is it that we can't use them straight away inside Angular? Well, the thing is that not every browser support
the Promise object yet ([It is looking good, though]).

But lucky us, there is a hole bunch of libraries out there that polyfills promises, happy days! Angular has it's own implementation of promises and it is based on
an existing library: [Kris Kowal's q]

The $q service
==============
There are two ways we can use this service. ES6-ish way or jQuery's deferred way. I personally recommend the ES6-ish way because it is more similar to the specification
than the other one and it is more readable IMHO. Let's see both of them, even though I will stick to the first one for the rest of the post.

{% highlight js %}
//ES6-ish way
var es6Promise = $q(function(resolve, reject){
  var result = doWhatever();
  if(result){
    resolve('OK');
  }else{
    reject('KO');
  }
});

//Deferred way
var deferredPromise = function(){
  var deferred = $q.defer(),
      result = doWhatever();

  if(result){
    deferred.resolve('OK');
  }else{
    deferred.reject('KO');
  }
  return deferred.promise;
}();

{% endhighlight %}

Both versions are ok, I said my opinion, so it is your turn to stick to whatever you like the most.

From here it is the same as with standard Promises. You can chain **then**s, handle errors with **catch**
or join multiple promises with **$q.all(promises)**. Don't forget you can create rejected or fulfilled 
promises with **$q.reject()** and **$q.resolve()**.

Conclusion
-----------

It is your turn to start using this and keepeing your code nice and tidy. Forget about callback hell and all that
stuff, promises are here and they are for good. 

Enjoy!


[official specification]: http://people.mozilla.org/~jorendorff/es6-draft.html#sec-promise-objects
[It is looking good, though]: http://caniuse.com/#feat=promises
[Kris Kowal's q]: https://github.com/kriskowal/q

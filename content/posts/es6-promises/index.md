---
title: "Exploring ES6 Promises"
date: 2017-03-19T21:13:10+02:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - Web Development
tags:
  - ES6
  - Javascript
  - NodeJS
---

So I've been reading up on more ES6 features lately, and most recently Promises. [Ben Ilegbodu](www.benmvp.com), someone I very much look up to advised that a good way to learn programming concepts is to either implement in a project, or blog about it - I challenged myself to doing both. Here are my thoughts on what I've learnt so far.

## ES6 Promises.
Promises are a Javascript pattern for the delivery of the result of an asynchronous computation. They provide a better way for working with callbacks. (and are a likely future replacement).

Below is a basic example of a Promise implementation:
    <!--more-->
{% highlight text %}
const asyncFunc = (someArgs) => {
  return new Promise((resolve, reject) => {
    //some code
    if(condiition) {
      resolve(args)
    } else {
      //some other code
      reject(error)
    }
  }
}
{% endhighlight %}

We can then invoke the above like this:
{% highlight text %}
asyncFunc()
  .then((result) => {
    //some code
    return asyncFunc2();
  ))
  .then(() => {
    //some code
  })
  .catch((error) => {
    //handle error here
  });
{% endhighlight %}


## Chaining Promises.
Given that ***P.then((optionalParam) => {...})*** returns a new promises Q, we can continue the promise-based flow by invoking then() on Q.

Q is resolved with the results of either onFulfilled or onRejected
Q is rejected if either onFulfilled or onRejected throws an exception.

To achieve chaining, we can resolve Q by:
- returning a normal value
- return a thenable

If we resolve Q with a normal value, we can pick this value up via a subsequent then(), as in:

{% highlight text %}
p.then((returnedData) => {
  //Q here
  return 'testvalue';
})
.then((val) => {
  console.log(val); // ouput=> 'testvalue'
});
{% endhighlight %}

We can also resolve Q with a thenable - any object that has a then() function, that behaves like ***Promise.pototype.then()***. If we return a thenable from inside Q, we forward Q's resolution to the next chain (we'll call it 'R'). We'll have a chain that looks like this:

{% highlight text %}
p.then((returnedData) => {
  //Q here
  return thenableX()
})
.then((param) => {
  //sample code
  ...
});
{% endhighlight %}

**Note:** Its important to note here that any error that occurs within the then() methods in a Promise chain are passed on to the error handler (the catch block) if there is one.

## Executing Asynchronous functions in parallel.
When asynchronous functions are chained via the then() construct (as sited in the previous example), they are executed sequentially (i.e one after the other).
However, there are use-cases when we need the async functions to run in parallel, i.e:

{% highlight text %}
asyncFunc1();
asyncFunc2();
{% endhighlight %}

### Promise.all
With [Promise.all()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise/all), we get notified once all the specified async functions return have returned respective results.

Promise.all takes an array of promises as argument and returns a single promise that is fulfilled with an array of the results of each specified promise, i.e:

{% highlight text %}
Promise.all([
  asyncFunc1(),
  asyncFunc2(),
])
.then((...results)) => {
  //do something with array of results
}
.catch((error) => {
  ...
});
{% endhighlight %}

### Promise.race
[Promise.race()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise/race) takes an iterable of promises, and returns a promise that resolves or rejects as soon as any of the promises in the iteralbe resolves or rejects.

It'll look somewhat like this:
{% highlight text %}
Promise.race([
  asyncFunc1(),
  asyncFunc2(),
])
.then((fulfilmentResult)) => {
  //do something with
}
.catch((error) => {
  ...
});
{% endhighlight %}

## Promise States
A Promise can exist in one of the following three mutually exclusive states.

#### 1.Pending
This is the initial state of the promise, i.e when its results have not yet been computed.

#### 2. Fulfilled
In this state, the promise has been completed computation with a result.

#### 3. Rejected
The promise began computation, but ended with an error.

A promise is said to be settled(TOD::Bolden) when it's either fulfilled or rejected, Once settled, its state remains unchanged.

### Promise Reactions
These are the callbacks that are registered with the then() method(s) of a promise, to notify of its fulfillment or rejection.

## Advantages of Promises over Callbacks
#### No Inversion of Control
With Promises, there is no inversion of control. i.e Promise-based functions return results like synchronous code; they don't directly continue and control execution, thus giving the caller more control.

#### Escaping the callback-hell
With Promises, chaining of async code is possible, as compared to nesting when using callbacks. When the return value of the async code is a thenable*(TODO::Bolden), the then() function can then be used to chain more async code, for example:

{% highlight text %}
asyncFunc()
  .then((result) => {
    //some code
  ))
  .catch((error) => {
    //handle error here
  });
{% endhighlight %}

#### Error Handling
With promises, errors and exceptions are handled in a more IoC manner - they are passed down till a catch() block is reached in the Promise chain.

#### Standardization
Before promises, there were several ways of implementing asynchronous code. The Promise API adheres to the Promise/A+ standard.

Promises are operated from both a Producer and Consumer perspective.

### Creating a Promise
As a producer, a promise is created and its result set via:

{% highlight text %}
return new Promise((resolve, reject) => {
  //some code
  if(condiition) {
    resolve(args)
  } else {
    //some other code
    reject(error)
  }
}
{% endhighlight %}

In the code-block above, the parameter to the Promise constructor us called an executor(TODO:Bolden). If an exception is thrown with this executor, the promise transits into a rejected state.

Other ways of creating Promise include:

#### Promise.resolve()
This creates an immediately Fulfilled promise, e.g:

{% highlight text %}
Promise.resolve(x)
  .then((val) => {
    console.log(val); //x
  });
{% endhighlight %}

Note: In the code above, if x is a thenable, it is returned unchanged and is settled in the succeeding then().

#### Promise.reject()
This creates an immediately rejected promise.

{% highlight text %}
Promise.reject(error)
  .catch((val) => {
    console.log(val); //x
  });
{% endhighlight %}

### Consuming a Promise
Code that consumers a promise its notified about its state transitions via reactions: Pending -> Rejected / Pending -> Fulfilled (as we described earlier).

These reactions are passed as callbacks to the then() and catch() methods,
Reactions that are registered in a promise's pending state are notified when it is settled, while those registered after it is settled receive the cached settled value immediately.

### Common mistakes to avoid when chaining promises
Truthfully, I had started implementing promises in my projects before i read these up! Your guess was right.. i had to go back to refactor my code :) I think they are really helpful.

#### 1. Loosing the tail of a promise chain

{% highlight text %}
#### BAD
function incorrect() {
  const myPromise = P; //where P is a thenable
  myPromise.then((val) => {
    ...
  });

  return myPromise; //the value from the method chained to `myPromise` is never returned
}
{% endhighlight %}

{% highlight text %}
#### GOOD
function correct() {
  const myPromise = P; //where P is a thenable
  return myPromise.then((val) => {
    ...
  });
  //this way, the tail of myPromise's chain is returned upon resolution or rejection.
}
{% endhighlight %}

{% highlight text %}
#### BEST
function correct() {
  //we dont actually need to store the promise in a variable
  return P.then((val) => {
    ...
  });
  //this way, the tail of myPromise's chain is returned upon resolution or rejection.
}
{% endhighlight %}

#### 2. Creating Promises instead of chaining

{% highlight text %}
#### BAD
const doSomething = () => {
  return new Promise(function(resolve, reject) {

    fetch('http://www.google.com')
      .then((response) => {
        resolve(successResponse);
      }).catch((error) => {
        reject(error)
      });
  });
};
{% endhighlight %}

{% highlight text %}
#### GOOD
const doSomething = () => {
  return fetch('http://www.google.com')
    .then((response) => {
      return successResponse;
    });
};
{% endhighlight %}

#### 3. Using then() for error handling

{% highlight text %}
#### BAD
doSomethingAsync()
  .then(
    (response) => { --> A
      //some code here
      doSomething(response); ---> B
      return doSomethingAsync2(response); ---> C
    },
    (error) => {
      //handle error
    }
  );
{% endhighlight %}

This approach is wrong because there are a number of exception cases that aren't handled:
- Rejections created by the fulfillment callback in the code-block starting at (A)
- Exceptions thrown during execution of method at (B)
- Rejection due to execution of async code at (C)

{% highlight text %}
#### GOOD
  doSomethingAsync()
    .then(
      (response) => { --> A
        //some code here
        doSomething(response); ---> B
        return doSomethingAsync2(response); ---> C
      })
      .catch((error) => {
        //handle error
      });
{% endhighlight %}


I hope i've been able to shed some light on how to use ES6 promises. In the Part 2 of this article, we'll discuss some advanced Promise concepts such as:
- Composing Promises
- Using the Promise methods: done() and finally()
- Advantages and limitations of Promises


### Additional Resources:
* [Learning ES6 Promises by Ben Ilegbodu](http://www.benmvp.com/learning-es6-promises)
* [Exploring JS - 25. Promises for asynchronous programming](http://exploringjs.com/es6/ch_promises.html)
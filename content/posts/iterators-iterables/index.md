---
title: "ES6 Iterators and Iterables: Salient Points"
date:  2017-02-19T19:28:31+02:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - ES6
  - Web Development
  - NodeJS
tags:
  - Javascript
  - Node
---

Iterators are ES6's new way of traversing data.

## Iterable.
An Iterable is a data structure that makes its elements publicly available by implementing a method whose key is `[Symbol.Iterator]`.

The following built-in types are iterables:
<!--more-->
- String
- Array
- Set
- Map
- DOM Data Structures.

## Iterator.
An iterator is a pointer for traversing the elements of a data structure. An Object is an Iterator when it implements the **next** method with the following properties:

* **done** <br>
  This is a boolean-typed property.
  It has a value of *true* if the iterator is past the end of the sequence(data structure) being iterated.

* **value** <br>
  This holds the value returned by the current iteration of the sequence. Its can be ommitted when *done* is true.


## Iterability.
Javascript introduced the Iterable interface ( Given that JS doesn't have actual interfaces, this is more of a convention, validated by [Duck Typing](https://en.wikipedia.org/wiki/Duck_typing)).

The Iterability concept consists of two core concepts:

### Data Sources
These are objects that implement the iterable interface. these objects could be from built-in types (e.g Array, Set, Map) or user-defined.

### Data Consumers
JavaScript constructs that consume data using the Iteration Protocol (e.g for..loop iterating over Array elements). are described as data consumers. Other data consumers include:
- Destructuring via Array Constructors.
- Promise.all and Promise.race.
- for...of
- Array.from()
- Spread Operator
- yield*

## Implementing Iterables

The most convenient (and probably best) means of implementing iterables is via [Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator).

All major ES6 data structures have three(3) methods that return iterable objects, as follows:
- entries()
- keys()
- values()

* **N.B** *Its important to note that plain Javascript objects are not iterable.*


## **Optional Iterator Methods** (*return* and *throw*)

### return()
This gives the iterator an opportuniy to auto-cleanup should an iteration end prematurely, after which it closes the iterator.

### throw()
When used, this forwards a method call to a generator that is iterated over via `yield`. A [further study on iterators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#iterator) would expatiate on this.


## Finishing Iterators
The iteration protocols describes two ways of finishing an iterator:
- Exhaustion
- Closing

### Exhaustion
This is the more common approach to finishing iterators. Exhaustion is as a result of complete retrieval of the eleements of the data structure over which an iterator is traversing.

### Closing
This is usually achieved by calling **return()** within the iteration code. By doing this, we tell the iterator not to make another call to its next() method.

Also, closure of an iterator can also be achieved by calling any of the following:
- break
- throw
- continue.

One major rule when closing an iterator with *return()* is:
> An iterator's return method should only be called before it is exhausted.


### Article References:
* [Iterators and Iterables by Ben Ilegbodu](http://www.benmvp.com/learning-es6-iterators-iterables)
* [Exploring JS - 21. Iterables and iterators](http://exploringjs.com/es6/ch_iteration.html)

---
title: 7 Exciting New JavaScript Features You Need to Know
date: "2019-08-05T00:00:00.000Z"
description: ""
---

This article has been translated to Japanese by [@rana_kualu](https://twitter.com/rana_kualu) here [https://qiita.com/rana_kualu/items/ee7694aa1cd4ae7f4483](https://qiita.com/rana_kualu/items/ee7694aa1cd4ae7f4483)

JavaScript (or ECMA Script) is an evolving language with lots of proposals and ideas on how to move forward. TC39 (Technical Committee 39) is the committee responsible for defining JS standards and features, and they have been quite active this year. Here is a summary of some proposals that are currently in "Stage 3", which is the last stage before becoming "finished". This means that these features should be implemented in browsers and other engines pretty soon. In fact, some of them are available now.

## 1. Private fields `#`
*Available in Chrome & NodeJS 12*
    
Yes, you read that right. Finally, JS is getting private fields in classes. No more `this._doPrivateStuff()`, defining closures to store private values, or using `WeakMap` to hack private props.

![don't touch my garbage](https://www.meme-arsenal.com/memes/74402a52240be627fb62e298b1fe0897.jpg)

Here's how the syntax looks

```javascript
// private fields must start with '#'
// and they can't be accessed outside the class block

class Counter {
  #x = 0;

  #increment() {
    this.#x++;
  }

  onClick() {
    this.#increment();
  }

}

const c = new Counter();
c.onClick(); // works fine
c.#increment(); // error
```
   
Proposal: https://github.com/tc39/proposal-class-fields


## 2. Optional Chaining `?.`

Ever had to access a property nested a few levels inside an object and got the infamous error `Cannot read property 'stop' of undefined`. Then you change your code to handle every possible `undefined` object in the chain, like:

```javascript
const stop = please && please.make && please.make.it && please.make.it.stop;

// or use a library like 'object-path'
const stop = objectPath.get(please, "make.it.stop");

```

With optional chaining, soon you'll be able to get the same done writing:


```javascript
const stop = please?.make?.it?.stop;

```


Proposal: https://github.com/tc39/proposal-optional-chaining


## 3. Nullish Coalescing `??`

It's very common to have a variable with an optional value that can be missing, and to use a default value if it's missing

```javascript
const duration = input.duration || 500;
```

The problem with `||` is that it will override all falsy values like (`0`, `''`, `false`) which might be in some cases valid input.

Enter the nullish coalescing operator, which only overrides `undefined` or `null`

```javascript
const duration = input.duration ?? 500;
```


Proposal: https://github.com/tc39/proposal-nullish-coalescing


## 4. BigInt `1n`
*Available in Chrome & NodeJS 12*

One of the reasons JS has always been terrible at Math is that we can't reliably store numbers larger than `2 ^ 53`, which makes it pretty hard to deal with considerably large numbers. Fortunately, `BigInt` is a proposal to solve this specific problem.

![Trump: gonna be HUUUUUGE](https://i.imgflip.com/p8blw.jpg)


Without further ado

```javascript
// can define BigInt by appending 'n' to a number literal
const theBiggestInt = 9007199254740991n;

// using the constructor with a literal
const alsoHuge = BigInt(9007199254740991);

// or with a string
const hugeButString = BigInt('9007199254740991');

```

You can also use the same operators on `BigInt` as you would expect from regular numbers, eg: `+`, `-`, `/`, `*`, `%`, ... There's a catch though, you can't mix `BigInt` with numbers in most operations. Comparing `Number` and `BigInt` works, but not adding them


```javascript
1n < 2 
// true

1n + 2
// 🤷‍♀️ Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions
```

Proposal: https://github.com/tc39/proposal-bigint

## 5. `static` Fields
*Available in Chrome & NodeJS 12*

This one is pretty straightforward. It allows having static fields on classes, similar to most OOP languages. Static fields can be useful as a replacement for enums, and they also work with private fields.

```javascript
class Colors {
  // public static fields
  static red = '#ff0000';
  static green = '#00ff00';

  // private static fields
  static #secretColor = '#f0f0f0';

}


font.color = Colors.red;

font.color = Colors.#secretColor; // Error
```

Proposal: https://github.com/tc39/proposal-static-class-features


## 6. Top Level `await`
*Available in Chrome*

Allows you to use await at the top level of your code. This is super useful for debugging async stuff (like `fetch`) in the browser console without wrapping it in an async function.

![using await in browser console](https://thepracticaldev.s3.amazonaws.com/i/y5ur91fgud4pu7hh5ypv.png)

If you need a refresher on async & await, [check my article explaining it here](https://dev.to/gafi/7-reasons-to-always-use-async-await-over-plain-promises-tutorial-4ej9)

Another killer use case is that it can be used at the top level of ES modules that initialize in an async manner (think about your database layer establishing a connection). When such an "async module" is imported, the module system will wait for it to resolve before executing the modules that depend on it. This will make dealing with async initialization much easier than the current workarounds of returning an initialization promise and waiting for it. A module will not know whether its dependency is async or not.

![wait for it](http://24.media.tumblr.com/tumblr_m3x648wxbj1ru99qvo1_500.png)


```javascript
// db.mjs
export const connection = await createConnection();
```

```javascript
// server.mjs
import { connection } from './db.mjs';

server.start();
```

In this example, nothing will execute in `server.mjs` until the connection is complete in `db.mjs`.

Proposal: https://github.com/tc39/proposal-top-level-await

## 7. `WeakRef`
*Available in Chrome & NodeJS 12*

A weak reference to an object is a reference that is not enough to keep an object alive. Whenever we create a variable with (`const`, `let`, `var`) the garbage collector (GC) will never remove that variable from memory as long as its reference is still accessible. These are all strong references. An object referenced by a weak reference, however, may be removed by the GC at any time if there is no strong reference to it. A `WeakRef` instance has a method `deref` which returns the original object referenced, or `undefined` if the original object has been collected.

This might be useful for caching cheap objects, where you don't want to keep storing all of them in memory forever. 


```javascript

const cache = new Map();

const setValue =  (key, obj) => {
  cache.set(key, new WeakRef(obj));
};

const getValue = (key) => {
  const ref = cache.get(key);
  if (ref) {
    return ref.deref();
  }
};

// this will look for the value in the cache
// and recalculate if it's missing
const fibonacciCached = (number) => {
  const cached = getValue(number);
  if (cached) return cached;
  const sum = calculateFibonacci(number);
  setValue(number, sum);
  return sum;
};

```

This is probably not a good idea for caching remote data as it can be removed from memory unpredictably. It's better to use something like an LRU cache in that case.

Proposal: https://github.com/tc39/proposal-weakrefs

--------------

That's it. I hope you're as excited as I am to use these cool new features. For more details on these proposals and others that I didn't mention, [keep an eye on TC39 proposals on github](https://github.com/tc39/proposals)

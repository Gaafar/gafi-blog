---
title: 7 Reasons Why JavaScript Async/Await Is Better Than Plain Promises (Tutorial)
date: "2017-03-07T00:00:00.000Z"
description: ""
---

Async/await was introduced in NodeJS 7.6 and is currently supported in all modern browsers. I believe it has been the single greatest addition to JS since 2017. If you are not convinced, here are a bunch of reasons with examples why you should adopt it immediately and never look back.

### Async/Await 101

For those who have never heard of this topic before, here’s a quick intro

* Async/await is a new way to write asynchronous code. Previous alternatives for asynchronous code are callbacks and promises.
* Async/await is actually just syntax sugar built on top of promises. It cannot be used with plain callbacks or node callbacks.
* Async/await is, like promises, non-blocking.
* Async/await makes asynchronous code look and behave a little more like synchronous code. This is where all its power lies.

### Syntax

Assuming a function `getJSON` that returns a promise, and that promise resolves with some JSON object. We just want to call it and log that JSON, then return `"done"`.

This is how you would implement it using promises
`gist:Gaafar/5f10e86ab571dfdad7084c5df0663435`



And this is how it looks with async/await
`gist:Gaafar/30a2a05731e6dcfa786456d240a59c91`

There are a few differences here
1. Our function has the keyword `async` before it. The `await` keyword can only be used inside functions defined with `async`. Any `async` function returns a promise implicitly, and the resolve value of the promise will be whatever you `return` from the function (which is the string `"done"` in our case).

1. The above point implies that we can’t use `await` at the top level of our code since that is not inside an `async` function.
    `gist:Gaafar/c5f60d00d9923427a30db29ffedc6c9d`


1. `await getJSON()` means that the `console.log` call will wait until `getJSON()` promise resolves and print its value.

### Why Is It better?

1. Concise and clean
Look at how much code we didn’t write! Even in the contrived example above, it’s clear we saved a decent amount of code. We didn’t have to write `.then`, create an anonymous function to handle the response, or give a name `data` to a variable that we don’t need to use. We also avoided nesting our code. These small advantages add up quickly, which will become more obvious in the following code examples.

2. Error handling
Async/await makes it finally possible to handle both synchronous and asynchronous errors with the same construct, good old `try/catch`. In the example below with promises, the `try/catch` will not handle if `JSON.parse` fails because it’s happening inside a promise. We need to call `.catch` on the promise and duplicate our error handling code, which will (hopefully) be more sophisticated than `console.log` in your production-ready code.

    `gist:Gaafar/7661b4a6c5999505ee54cdefe9ad1c7a`
    
    Now look at the same code with async/await. The `catch` block now will handle parsing errors.

    `gist:Gaafar/65db187a0a7131d2499f3531c5614a33`

3. Conditionals
Imagine something like the code below which fetches some data and decides whether it should return that or get more details based on some value in the data.
    `gist:Gaafar/de1266d2338ba2393e5bada866178067`


    Just looking at this gives you a headache. It’s easy to get lost in all that nesting (6 levels), braces, and return statements that are only needed to propagate the final result up to the main promise.

    This example becomes way more readable when rewritten with async/await.

    `gist:Gaafar/e33b6f757793d439070b541437e69bd6`

4. Intermediate values
You have probably found yourself in a situation where you call a `promise1` and then use what it returns to call `promise2`, then use the results of both promises to call a `promise3`. Your code most likely looked like this

    `gist:Gaafar/8d8767915d4bf2225ee6f844fc525b47`

    If `promise3` didn’t require `value1` it would be easy to flatten the promise nesting a bit. If you are the kind of person who couldn’t live with this, you could wrap both values 1 & 2 in a `Promise.all` and avoid deeper nesting, like this

    `gist:Gaafar/25665bc0aa50118a315fddae935cda25`

    This approach sacrifices semantics for the sake of readability. There is no reason for `value1` & `value2` to belong in an array together, except to avoid nesting promises.
This same logic becomes ridiculously simple and intuitive with async/await. It makes you wonder about all the things you could have done in the time that you spent struggling to make promises look less hideous.

    `gist:Gaafar/d28c62ad9a92b8e4ecf7b34e253d4dd8`

5. Error stacks
Imagine a piece of code that calls multiple promises in a chain, and somewhere down the chain, an error is thrown.

    `gist:Gaafar/39b83dccf61ff9e86c5e31e297d96300`

    The error stack returned from a promise chain gives no clue of where the error happened. Even worse, it’s misleading; the only function name it contains is `callAPromise` which is totally innocent of this error (the file and line number are still useful though).
However, the error stack from async/await points to the function that contains the error

    `gist:Gaafar/f01a1ee86ca9937dfcdcfcbac45c6a4f`

    This is not a huge plus when you’re developing on your local environment and have the file open in an editor, but it’s quite useful when you’re trying to make sense of error logs coming from your production server. In such cases, knowing the error happened in `makeRequest` is better than knowing that the error came from a `then` after a `then` after a `then` …

6. Debugging
A killer advantage when using async/await is that it’s much easier to debug. Debugging promises has always been such a pain for 2 reasons
    1. You can’t set breakpoints in arrow functions that return expressions (no body).

        ![snippet with chained promises](https://miro.medium.com/max/1260/1*n_V4LaVdBOFgGCbmTR_VKA.png)*Try setting a breakpoint anywhere here*


    2. If you set a breakpoint inside a `.then` block and use debug shortcuts like step-over, the debugger will not move to the following `.then` because it only “steps” through synchronous code.

        With async/await you don’t need arrow functions as much, and you can step through await calls exactly as if they were normal synchronous calls.

        ![snippet with consecutive awaits](https://miro.medium.com/max/1260/1*GWYd4eLrs0U96MkNNVB56A.png)

7. You can `await` anything
Last but not least, `await` can be used for both synchronous and asynchronous expressions. For example, you can write `await 5`, which is equivalent to `Promise.resolve(5)`. This might not seem very useful at first, but it's actually a great advantage when writing a library or a utility function where you don't know whether the input will be sync or async. 

    Imagine you want to record the time taken to execute some API calls in your application, and you decide to create a generic function for this purpose. Here's how it would look with promises
    `gist:Gaafar/de12a44a03ef9aa1028fbbc6f58f5d3c`
    
    You know that all API calls are going to return promises, but what happens if you use the same function to record the time taken in a synchronous function? It will throw an error because the sync function does not return a promise. The usual way to avoid this is wrapping `makeRequest()` in `Promise.resolve()`

    If you use async/await, you won't have to worry about these cases because await allows you to work safely with any value, promise or not.
    `gist:Gaafar/14feecc0965af13c9c5447e388c2717f`


### In Conclusion

Async/await is one of the most revolutionary features that have been added to JavaScript in the past few years. It makes you realize what a syntactical mess promises are, and provides an intuitive replacement.

### Concerns

Some valid skepticism you might have about using async/await is that it makes asynchronous code less obvious: Our eyes learned to spot asynchronous code whenever we see a callback or a `.then`, it will take a few weeks for your eyes to adjust to the new signs, but C# had this feature for years and people who are familiar with it know it’s worth this minor, temporary inconvenience.

Follow me on twitter [@imgaafar](https://twitter.com/imGaafar)

This article was originally [published on Hackernoon](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9)
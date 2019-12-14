# Promise.all()

The `Promise.all()` method takes multiple promises passed as an iterable and returns a single Promise that:
  - fulfills when:
    - all promises have been fulfilled.
    - the iterable contains no promises.
  - rejects with the reason of the first promise that rejects.

Typical use case: For when we have started multiple asynchronous tasks to run concurrently and want to wait for all the tasks being finished.

## Syntax

`Promise.all(iterable);`

### Parameters

An iterable object suchs as an `Array`.

### Return Value

- If the iterable is empty: An **already resolved** `Promise`.
- If the iterable contains no promises: An **ansynchronously resolved** `Promise`.
- In all other cases: A pending `Promise` in all other cases.

## Description

This method can be useful for aggregating the results of multiple promises.

### Fulfillment

The returned promise is fulfilled with an array of all values, including non-promises values, in the iterable argument.

  - If the iterable is empty, returns (synchronously) an already resolved promise.
  - If all of the promises fulfill, or there are no promises, returns a promise that fulfills asynchronously.

### Rejection

If any of the passed-in promises reject, returns a promise that rejects asynchronously with the value of the promise that rejected, whether or not the other promises have resolved.

## Examples

### Using `Promise.all`

`Promise.all` waits for all fulfillments or the first rejection.

    var promise1 = Promise.resolve(3);
    var promise2 = 42;
    var promise3 = new Promise(function(resolve, reject) {
      setTimeout(resolve, 100, 'foo');
    });

    Promise.all([promise1, promise2, promise3])
      .then(function(values) {
        console.log(values) // [3, 42, 'foo']
      });

If the iterable contains non-promise values, they will be ignored, but still counted in the returned promise array value (if the promise is fulfilled):

    // treated as if the iterable is empty, and gets fulfilled
    var p = Promise.all([1, 2, 3]);

    // treated as if the iterable contains only the resolved promise with value "444", and gets fulfilled
    var p2 = Promise.all([1, 2, 3, Promise.resolve(444)]);

    // treated as if the iterable passed contains only the rejected promise with value "555", and gets rejected
    var p3 = Promise.all([1, 2, 3, Promise.reject(555)]);

    // using setTimeout we can execute code after the stack is empty
    setTimeout(function() {
        console.log(p); // Promise { <state>: "fulfilled", <value>: Array[3] }
        console.log(p2); // Promise { <state>: "fulfilled", <value>: Array[4] }
        console.log(p3); // Promise { <state>: "rejected", <reason>: 555 }
    });

## Asynchronicity or synchronicity of `Promise.all`

This following example demonstrates the asynchronicity (or synchronicity, if the iterable passed is empty) of `Promise.all`:

    // we are passing as argument an array of promises that are already resolved, to trigger Promise.all as soon as possible
    var resolvedPromisesArray = [Promise.resolve(33), Promise.resolve(44)];

    var p = Promise.all(resolvedPromisesArray);

    // immediately logging the value of p
    console.log(p);

    // using setTimeout we can execute code after the stack is empty
    setTimeout(function() {
      console.log('the stack is now empty');
      console.log(p);
    });

    // From console:
    // Promise { <state>: "pending" } 
    // the stack is now empty
    // Promise { <state>: "fulfilled", <value>: Array[2] }

The same thing happens if Promise.all rejects:

    var mixedPromisesArray = [Promise.resolve(33), Promise.reject(44)];
    var p = Promise.all(mixedPromisesArray);
    
    console.log(p);
    
    setTimeout(function() {
        console.log('the stack is now empty');
        console.log(p);
    });

    // From console:
    // Promise { <state>: "pending" } 
    // the stack is now empty
    // Promise { <state>: "rejected", <reason>: 44 }
    
But, Promise.all resolves synchronously **if and only if** the iterable passed is empty:

    // will be immediately resolved
    var p = Promise.all([]);

    // non-promise values will be ignored, but the evaluation will be done asynchronously
    var p2 = Promise.all([1337, "hi"]); 

    console.log(p);
    console.log(p2)

    setTimeout(function() {
        console.log('the stack is now empty');
        console.log(p2);
    });

    // From console:
    // Promise { <state>: "fulfilled", <value>: Array[0] }
    // Promise { <state>: "pending" }
    // the stack is now empty
    // Promise { <state>: "fulfilled", <value>: Array[2] }

## `Promise.all` fail-fast behaviour

`Promise.all` is rejected immediately if any of the elements are rejected.

    var p1 = new Promise((resolve, reject) => { 
      setTimeout(() => resolve('one'), 1000); 
    });

    var p2 = new Promise((resolve, reject) => { 
      setTimeout(() => resolve('two'), 2000); 
    });

    var p3 = new Promise((resolve, reject) => {
      setTimeout(() => resolve('three'), 3000);
    });

    var p4 = new Promise((resolve, reject) => {
      setTimeout(() => resolve('four'), 4000);
    });

    var p5 = new Promise((resolve, reject) => {
      reject(new Error('reject'));
    });

    // Using .catch:
    Promise.all([p1, p2, p3, p4, p5])
    .then(values => { 
      console.log(values);
    })
    .catch(error => { 
      console.error(error.message)
    });

    // From console: 
    // "reject"

It is possible to change this behaviour by handling possible rejections:

    var p1 = new Promise((resolve, reject) => { 
      setTimeout(() => resolve('p1 delayed resolution'), 1000); 
    }); 

    var p2 = new Promise((resolve, reject) => {
      reject(new Error('p2 immediate rejection'));
    });

    Promise.all([
      p1.catch(error => { return error }),
      p2.catch(error => { return error }),
    ]).then(values => { 
      console.log(values[0]) // "p1 delayed resolution"
      console.error(values[1]) // "Error: p2 immediate rejection"
    });
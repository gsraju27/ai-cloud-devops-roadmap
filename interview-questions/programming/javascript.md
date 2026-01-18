# Complete JavaScript Interview Guide

Master JavaScript with these real-world interview questions covering core concepts, asynchronous programming, and data structures. Practice scenarios that mirror actual frontend and backend engineering challenges.

**Companies that ask these questions:** Google | Facebook | Netflix | Microsoft | Amazon

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What is the event loop and how does it work in JavaScript? | Conceptual | Core Concepts |
| 2 | What is the difference between `==` and `===` in JavaScript? | Conceptual | Core Concepts |
| 3 | What are closures and how do you use them in JavaScript? | Conceptual | Core Concepts |
| 4 | What is the `this` keyword and how does it work in JavaScrip... | Conceptual | Core Concepts |
| 5 | What is the prototype chain and how does it work in JavaScri... | Conceptual | Object-Oriented Programming |
| 6 | What are Promises and how do you use them in JavaScript? | Conceptual | Asynchronous Programming |
| 7 | What are `async/await` and how do you use them in JavaScript... | Conceptual | Asynchronous Programming |
| 8 | What is the difference between `let`, `const`, and `var` in ... | Conceptual | Core Concepts |
| 9 | What are the different data types in JavaScript? | Conceptual | Core Concepts |
| 10 | What are the different ways to create an object in JavaScrip... | Conceptual | Object-Oriented Programming |
| 11 | What are the different ways to handle asynchronous operation... | Conceptual | Asynchronous Programming |
| 12 | What is the difference between `null` and `undefined` in Jav... | Conceptual | Core Concepts |

---

## What You'll Learn

- Understand the core concepts of JavaScript.
- Master asynchronous programming.
- Work with data structures and algorithms.
- Write object-oriented and functional code.
- Debug and optimize JavaScript applications.

---

## Interview Questions

### Question 1: What is the event loop and how does it work in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new feature that needs to make an API call to get some data and then update the UI with the data.

You need to understand how the event loop works so that you can write asynchronous code that is both performant and correct.

## The Challenge

Explain what the event loop is in JavaScript and how it works. What is the role of the call stack, the message queue, and the microtask queue?


> **Common Mistake:** A junior engineer might have a vague understanding of the event loop. They might not be able to explain the difference between the message queue and the microtask queue, or the implications of this difference.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the event loop and its different components. They would also be able to explain the difference between the message queue and the microtask queue and would have a clear understanding of how this affects the execution of asynchronous code.

### Step 1: Understand the Key Components

| Component          | Description                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| **Call Stack**     | A LIFO (Last-In, First-Out) data structure that stores the execution context of the code.                 |
| **Message Queue**  | A FIFO (First-In, First-Out) data structure that stores the messages to be processed by the event loop. |
| **Microtask Queue**| A queue for microtasks, such as promise callbacks. Microtasks are executed before the next message in the message queue. |
| **Event Loop**     | A process that constantly checks the call stack and the message queue. If the call stack is empty, it takes the first message from the message queue and pushes it onto the call stack. |

### Step 2: The Event Loop in Action

Here's how the event loop works:

1.  When a function is called, it is pushed onto the call stack.
2.  When the function returns, it is popped off the call stack.
3.  When an asynchronous operation is performed, it is offloaded to the browser's Web API.
4.  When the asynchronous operation is complete, a message is added to the message queue.
5.  The event loop takes the first message from the message queue and pushes its callback function onto the call stack.
6.  The callback function is executed.

### Microtasks vs. Macrotasks

The message queue is also known as the **macrotask queue**. In addition to the macrotask queue, there is also a **microtask queue**.

-   **Macrotasks:** `setTimeout`, `setInterval`, `setImmediate`
-   **Microtasks:** `process.nextTick`, `Promise.then`, `queueMicrotask`

Microtasks are always executed before the next macrotask.


---

### Quick Check

**You have a `setTimeout` with a delay of 0ms and a `Promise.then`. Which one will be executed first?**

   A. The `setTimeout`
-> B. **The `Promise.then`**
   C. It depends on the browser.
   D. It's not possible to know.

<details>
<summary>See Answer</summary>

The `Promise.then` will be executed first. This is because promise callbacks are microtasks, and microtasks are always executed before the next macrotask.

</details>

---

### Question 2: What is the difference between `==` and `===` in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to compare two values to see if they are equal.

You are not sure whether to use the `==` operator or the `===` operator.

## The Challenge

Explain the difference between the `==` and `===` operators in JavaScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference between type coercion and strict equality.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `==` and `===`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Operator     | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **`==`**     | The loose equality operator. It compares two values for equality, after converting both values to a common type. |
| **`===`**    | The strict equality operator. It compares two values for equality, without any type conversion.         |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use the **`===` operator**. This is because it is more predictable and less error-prone than the `==` operator.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`==`:**

```javascript
console.log(1 == '1'); // true
console.log(true == 1); // true
console.log(null == undefined); // true
```

**`===`:**

```javascript
console.log(1 === '1'); // false
console.log(true === 1); // false
console.log(null === undefined); // false
```

### When to use `==`

You should almost never use the `==` operator. The only exception is when you want to check if a value is `null` or `undefined`.

```javascript
if (myVar == null) {
  // This will be true if myVar is null or undefined.
}
```

---

### Question 3: What are closures and how do you use them in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to create a private variable that can only be accessed by a specific function.

## The Challenge

Explain what closures are in JavaScript and how you would use them to solve this problem. What are the key benefits of using closures?


> **Common Mistake:** A junior engineer might not be aware of closures. They might try to solve this problem by using a global variable, which would not be a very good solution.

> **Senior Engineer Approach:** A senior engineer would know that closures are the perfect tool for this job. They would be able to explain what closures are and how to use them to create private variables.

### Step 1: Understand What Closures Are

A closure is a function that has access to the variables in its outer (enclosing) function's scope chain. In other words, a closure allows a function to access variables from an outer function, even after the outer function has returned.

### Step 2: Create a Private Variable

Here's how we can use a closure to create a private variable:

```javascript
function createCounter() {
  let count = 0;

  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

In this example, the `createCounter` function returns a new function. This new function has access to the `count` variable from the `createCounter` function's scope, even after the `createCounter` function has returned.

### The Benefits of Using Closures

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Data Encapsulation** | Closures allow you to create private variables that can only be accessed by a specific function.     |
| **Stateful Functions**| Closures allow you to create stateful functions that can remember their state between calls.           |
| **Partial Application**| Closures can be used to create partially applied functions.                                         |

---

### Question 4: What is the `this` keyword and how does it work in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new component that has a method that needs to access a property of the component.

You are not sure how to use the `this` keyword to access the property.

## The Challenge

Explain what the `this` keyword is in JavaScript and how its value is determined. What are the different ways to set the value of `this`, and what are the common pitfalls to avoid?


> **Common Mistake:** A junior engineer might be confused by the `this` keyword. They might not be aware of the different ways to set its value, or they might not know how to use it correctly in different contexts.

> **Senior Engineer Approach:** A senior engineer would have a deep understanding of the `this` keyword. They would be able to explain how its value is determined and would have a clear plan for how to use it correctly in different contexts.

### Step 1: Understand How the Value of `this` is Determined

The value of the `this` keyword is determined by how a function is called.

| How the function is called | Value of `this`                                                                                             |
| -------------------------- | --------------------------------------------------------------------------------------------------------- |
| **In the global context**  | The global object (`window` in a browser, `global` in Node.js).                                         |
| **As a method of an object**| The object that the method was called on.                                                                 |
| **With `new`**             | A new object that is created by the `new` operator.                                                     |
| **With `call`, `apply`, or `bind`** | The object that is passed as the first argument to `call`, `apply`, or `bind`. |
| **As an event listener**   | The element that the event was fired on.                                                                  |
| **As an arrow function**   | The value of `this` from the enclosing lexical scope.                                                      |

### Step 2: Code Examples

Here are some code examples that show how the value of `this` is determined:

**As a method of an object:**

```javascript
const myObject = {
  myMethod() {
    console.log(this);
  }
};

myObject.myMethod(); // myObject
```

**With `call`, `apply`, or `bind`:**

```javascript
function myFunction() {
  console.log(this);
}

myFunction.call({ a: 1 }); // { a: 1 }
```

**As an arrow function:**

```javascript
const myObject = {
  myMethod() {
    const myArrowFunction = () => {
      console.log(this);
    };
    myArrowFunction();
  }
};

myObject.myMethod(); // myObject
```


---

### Quick Check

**What will be logged to the console?**

-> A. **`window` or `global`**
   B. `myObject`
   C. `undefined`
   D. `TypeError`

<details>
<summary>See Answer</summary>

When a function is called in the global context, the value of `this` is the global object (`window` in a browser, `global` in Node.js).

</details>

---

### Question 5: What is the prototype chain and how does it work in JavaScript?

**Type:** Conceptual | **Category:** Object-Oriented Programming

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to create a new object that inherits properties from another object.

You are not sure whether to use classical inheritance or prototypal inheritance.

## The Challenge

Explain what the prototype chain is in JavaScript and how it works. What are the pros and cons of prototypal inheritance, and how does it compare to classical inheritance?


> **Common Mistake:** A junior engineer might not be aware of the prototype chain. They might try to solve this problem by using classical inheritance, which is not the standard way to do inheritance in JavaScript.

> **Senior Engineer Approach:** A senior engineer would have a deep understanding of the prototype chain. They would be able to explain what it is and how it works. They would also be able to explain the pros and cons of prototypal inheritance and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand What the Prototype Chain Is

The prototype chain is a mechanism in JavaScript that allows an object to inherit properties from another object.

Every object in JavaScript has a prototype. When you try to access a property on an object, JavaScript will first look for the property on the object itself. If it does not find the property on the object, it will look for the property on the object's prototype. If it does not find the property on the prototype, it will look for the property on the prototype's prototype, and so on, until it reaches the end of the prototype chain.

### Step 2: How to Create a Prototype Chain

Here's how we can create a simple prototype chain:

```javascript
const animal = {
  isAlive: true
};

const dog = {
  bark() {
    console.log('woof');
  }
};

Object.setPrototypeOf(dog, animal);

console.log(dog.isAlive); // true
dog.bark(); // woof
```

In this example, we use the `Object.setPrototypeOf()` method to set the prototype of the `dog` object to the `animal` object.

### Prototypal Inheritance vs. Classical Inheritance

| Feature      | Prototypal Inheritance                                                              | Classical Inheritance                                                              |
| ------------ | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Inheritance**| Objects inherit from other objects.                                                | Classes inherit from other classes.                                                |
| **Flexibility**| More flexible, you can add or remove properties from an object at runtime.        | Less flexible, you cannot add or remove properties from a class at runtime.         |
| **Complexity** | Simpler and more intuitive.                                                       | More complex and verbose.                                                          |

### When to use Prototypal Inheritance

You should almost always use prototypal inheritance in JavaScript. It is the standard way to do inheritance in JavaScript, and it is more flexible and less complex than classical inheritance.

---

### Question 6: What are Promises and how do you use them in JavaScript?

**Type:** Conceptual | **Category:** Asynchronous Programming

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to make an API call to get some data and then update the UI with the data.

You need to find a way to handle the asynchronous nature of the API call.

## The Challenge

Explain what Promises are in JavaScript and how you would use them to solve this problem. What are the different states of a Promise, and what are the key methods that you would use to interact with a Promise?


> **Common Mistake:** A junior engineer might try to solve this problem by using a callback. This would work, but it would not be a very robust solution. They might not be aware of Promises, which are the standard way to handle asynchronous operations in modern JavaScript.

> **Senior Engineer Approach:** A senior engineer would know that Promises are the perfect tool for this job. They would be able to explain what Promises are and how to use them to handle asynchronous operations.

### Step 1: Understand What Promises Are

A `Promise` is an object that represents the eventual completion (or failure) of an asynchronous operation and its resulting value.

### Step 2: The Different States of a Promise

| State       | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **`pending`** | The initial state of a Promise.                                                                         |
| **`fulfilled`**| The state of a Promise that has been successfully resolved.                                             |
| **`rejected`**| The state of a Promise that has been rejected.                                                          |

### Step 3: The Key Methods of a Promise

| Method      | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **`then()`**  | Called when a Promise is fulfilled.                                                                     |
| **`catch()`** | Called when a Promise is rejected.                                                                      |
| **`finally()`**| Called when a Promise is settled (either fulfilled or rejected).                                        |

### Step 4: Solve the Problem

Here's how we can use a Promise to make an API call:

```javascript
fetch('https://api.example.com/my-data')
  .then(response => response.json())
  .then(data => {
    // ... (update the UI with the data) ...
  })
  .catch(error => {
    // ... (handle the error) ...
  });
```

In this example, the `fetch()` function returns a `Promise` that resolves with the response from the API. We then use the `then()` method to parse the response as JSON and to update the UI with the data. We use the `catch()` method to handle any errors that occur.


---

### Quick Check

**You want to make two API calls in parallel and then do something with the results of both calls. Which of the following would you use?**

-> A. **`Promise.all()`**
   B. `Promise.race()`
   C. `Promise.allSettled()`
   D. None of the above

<details>
<summary>See Answer</summary>

`Promise.all()` is the correct choice for this task. It will wait for both API calls to complete and then it will return an array with the results of both calls.

</details>

---

### Question 7: What are `async/await` and how do you use them in JavaScript?

**Type:** Conceptual | **Category:** Asynchronous Programming

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to make an API call to get some data and then update the UI with the data.

You are using Promises to handle the asynchronous nature of the API call, but you find the code to be a bit verbose and difficult to read.

## The Challenge

Explain what `async/await` is in JavaScript and how you would use it to solve this problem. What are the key benefits of using `async/await` over Promises?


> **Common Mistake:** A junior engineer might not be aware of `async/await`. They might just use Promises, which would work, but it would not be as readable as using `async/await`.

> **Senior Engineer Approach:** A senior engineer would know that `async/await` is the perfect tool for this job. They would be able to explain what `async/await` is and how to use it to write more readable and maintainable asynchronous code.

### Step 1: Understand What `async/await` Is

`async/await` is a feature that was added in ES2017. It is syntactic sugar on top of Promises that makes asynchronous code look and behave more like synchronous code.

### Step 2: The `async` and `await` Keywords

| Keyword   | Description                                                                                             |
| --------- | ------------------------------------------------------------------------------------------------------- |
| **`async`** | The `async` keyword is used to declare an asynchronous function.                                      |
| **`await`** | The `await` keyword is used to wait for a `Promise` to be resolved.                                   |

### Step 3: Solve the Problem

Here's how we can use `async/await` to make an API call:

```javascript
async function getData() {
  try {
    const response = await fetch('https://api.example.com/my-data');
    const data = await response.json();
    // ... (update the UI with the data) ...
  } catch (error) {
    // ... (handle the error) ...
  }
}
```

In this example, the `await` keyword is used to wait for the `fetch()` function to return a response and for the `response.json()` method to parse the response as JSON.

### `async/await` vs. Promises

| Feature      | Promises                                                              | `async/await`                                                              |
| ------------ | --------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Readability**| Can be difficult to read, especially for complex asynchronous operations. | Very readable, makes asynchronous code look and behave like synchronous code. |
| **Error Handling**| Uses the `.catch()` method to handle errors.                      | Uses a `try...catch` block to handle errors.                                |


---

### Quick Check

**You want to make two API calls in parallel and then do something with the results of both calls. Which of the following would you use?**

   A. `await` on each call one after the other
-> B. **`Promise.all()` with `await`**
   C. `Promise.race()` with `await`
   D. None of the above

<details>
<summary>See Answer</summary>

`Promise.all()` with `await` is the correct choice for this task. It will wait for both API calls to complete and then it will return an array with the results of both calls.

</details>

---

### Question 8: What is the difference between `let`, `const`, and `var` in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to declare a variable.

You are not sure whether to use `let`, `const`, or `var`.

## The Challenge

Explain the difference between `let`, `const`, and `var` in JavaScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in scope or the fact that `const` creates a read-only reference to a value.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `let`, `const`, and `var`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Key Differences

| Feature      | `var`                                                              | `let`                                                              | `const`                                                              |
| ------------ | ------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------- |
| **Scope**    | Function-scoped.                                                    | Block-scoped.                                                      | Block-scoped.                                                        |
| **Hoisting** | Hoisted and initialized with `undefined`.                           | Hoisted, but not initialized.                                      | Hoisted, but not initialized.                                        |
| **Re-declaration**| Can be re-declared.                                             | Cannot be re-declared in the same scope.                             | Cannot be re-declared in the same scope.                               |
| **Re-assignment** | Can be re-assigned.                                             | Can be re-assigned.                                                | Cannot be re-assigned.                                               |

### Step 2: Choose the Right Tool for the Job

In modern JavaScript, you should almost always use **`const`** by default. This is because it creates a read-only reference to a value, which can help to prevent bugs.

If you need to re-assign a variable, you should use **`let`**.

You should almost never use **`var`**.

### Step 3: Code Examples

Here are some code examples that show the difference between the three approaches:

**`var`:**

```javascript
function myFunction() {
  var x = 1;
  if (true) {
    var x = 2; // This will overwrite the previous x
    console.log(x); // 2
  }
  console.log(x); // 2
}
```

**`let`:**

```javascript
function myFunction() {
  let x = 1;
  if (true) {
    let x = 2;
    console.log(x); // 2
  }
  console.log(x); // 1
}
```

**`const`:**

```javascript
const x = 1;
// This will raise an error
// x = 2;
```


---

### Quick Check

**You need to declare a variable that will not be re-assigned. Which of the following would you use?**

   A. `var`
   B. `let`
-> C. **`const`**
   D. Any of the above

<details>
<summary>See Answer</summary>

`const` is the correct choice for this task. It creates a read-only reference to a value, which means that the variable cannot be re-assigned.

</details>

---

### Question 9: What are the different data types in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to work with a variety of different data types.

## The Challenge

Explain the different data types in JavaScript. What are the primitive data types, and what are the non-primitive data types?


> **Common Mistake:** A junior engineer might not be aware of all the different data types in JavaScript. They might also not be aware of the difference between primitive and non-primitive data types.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of all the different data types in JavaScript. They would also be able to explain the difference between primitive and non-primitive data types.

### Step 1: Understand the Primitive Data Types

| Data Type   | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **`string`**  | A sequence of characters.                                                                               |
| **`number`**  | A number, including integers and floating-point numbers.                                                 |
| **`boolean`** | A boolean value, either `true` or `false`.                                                              |
| **`null`**    | A special value that represents the intentional absence of any object value.                               |
| **`undefined`**| A special value that represents a variable that has been declared but has not yet been assigned a value. |
| **`symbol`**  | A unique and immutable value that can be used as the key of an object property.                         |
| **`bigint`**  | A number that can represent integers of arbitrary precision.                                            |

### Step 2: Understand the Non-Primitive Data Type

| Data Type  | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **`object`** | A collection of key-value pairs.                                                                        |

Arrays and functions are also objects in JavaScript.

### Step 3: The `typeof` Operator

You can use the `typeof` operator to check the type of a variable.

```javascript
typeof "hello" // "string"
typeof 123 // "number"
typeof true // "boolean"
typeof undefined // "undefined"
typeof null // "object" (this is a well-known bug in JavaScript)
typeof Symbol() // "symbol"
typeof 123n // "bigint"
typeof {} // "object"
```


---

### Quick Check

**What will `typeof null` return?**

   A. `null`
   B. `undefined`
-> C. **`object`**
   D. `TypeError`

<details>
<summary>See Answer</summary>

`typeof null` returns `object`. This is a well-known bug in JavaScript that has been around since the beginning of the language.

</details>

---

### Question 10: What are the different ways to create an object in JavaScript?

**Type:** Conceptual | **Category:** Object-Oriented Programming

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to create a new object.

You are not sure which of the different ways to create an object in JavaScript you should use.

## The Challenge

Explain the different ways to create an object in JavaScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might only be aware of one way to create an object, such as using an object literal. They might not be aware of the other ways to create an object or the trade-offs between them.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of all the different ways to create an object in JavaScript. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Different Ways to Create an Object

| Method          | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **Object Literal**| The simplest way to create an object.                                                                   |
| **`new` Keyword** | Creates a new object and sets its prototype to the constructor function's prototype.                  |
| **`Object.create()`**| Creates a new object with the specified prototype object and properties.                              |
| **`class` Keyword**| Syntactic sugar on top of prototypal inheritance.                                                       |

### Step 2: Choose the Right Tool for the Job

| Use Case                               | Recommended Method |
| -------------------------------------- | ------------------ |
| Creating a simple object               | Object Literal     |
| Creating an object with a constructor  | `new` Keyword      |
| Creating an object with a specific prototype | `Object.create()`  |
| Creating a complex object with methods | `class` Keyword    |

### Step 3: Code Examples

Here are some code examples that show the difference between the different approaches:

**Object Literal:**

```javascript
const myObject = {
  myProperty: 'myValue'
};
```

**`new` Keyword:**

```javascript
function MyClass() {
  this.myProperty = 'myValue';
}

const myObject = new MyClass();
```

**`Object.create()`:**

```javascript
const myPrototype = {
  myMethod() {
    console.log('hello');
  }
};

const myObject = Object.create(myPrototype);
```

**`class` Keyword:**

```javascript
class MyClass {
  constructor() {
    this.myProperty = 'myValue';
  }

  myMethod() {
    console.log('hello');
  }
}

const myObject = new MyClass();
```


---

### Quick Check

**You want to create a new object that inherits from another object. Which of the following would you use?**

   A. Object Literal
   B. `new` Keyword
   C. `Object.create()`
-> D. **All of the above**

<details>
<summary>See Answer</summary>

All of these can be used to create a new object that inherits from another object. `Object.create()` is the most direct way to do it, but you can also use the `new` keyword with a constructor function or the `class` keyword.

</details>

---

### Question 11: What are the different ways to handle asynchronous operations in JavaScript?

**Type:** Conceptual | **Category:** Asynchronous Programming

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to make an API call to get some data and then update the UI with the data.

You are not sure which of the different ways to handle asynchronous operations in JavaScript you should use.

## The Challenge

Explain the different ways to handle asynchronous operations in JavaScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might only be aware of callbacks. They might not be aware of Promises or `async/await`, or the trade-offs between them.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of all the different ways to handle asynchronous operations in JavaScript. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Different Ways to Handle Asynchronous Operations

| Method          | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **Callbacks**   | A function that is passed as an argument to another function and is executed when the other function has completed. |
| **Promises**    | An object that represents the eventual completion (or failure) of an asynchronous operation.             |
| **`async/await`**| Syntactic sugar on top of Promises that makes asynchronous code look and behave more like synchronous code. |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use **`async/await`**. This is because it is the most modern and readable way to write asynchronous code in JavaScript.

### Step 3: Code Examples

Here are some code examples that show the difference between the three approaches:

**Callbacks:**

```javascript
function getData(callback) {
  setTimeout(() => {
    callback(null, 'some data');
  }, 1000);
}

getData((err, data) => {
  if (err) {
    // ... (handle the error) ...
  } else {
    // ... (use the data) ...
  }
});
```

**Promises:**

```javascript
function getData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('some data');
    }, 1000);
  });
}

getData()
  .then(data => {
    // ... (use the data) ...
  })
  .catch(error => {
    // ... (handle the error) ...
  });
```

**`async/await`:**

```javascript
async function getData() {
  const data = await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('some data');
    }, 1000);
  });
  // ... (use the data) ...
}
```


---

### Quick Check

**You want to make two API calls in parallel and then do something with the results of both calls. Which of the following would you use?**

   A. Callbacks
   B. Promises with `Promise.all()`
   C. `async/await` with `Promise.all()`
-> D. **All of the above**

<details>
<summary>See Answer</summary>

All of these can be used to solve this problem. However, `async/await` with `Promise.all()` is the most modern and readable way to do it.

</details>

---

### Question 12: What is the difference between `null` and `undefined` in JavaScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are debugging a piece of code and you see that a variable has the value `undefined`. You are not sure what this means or how it is different from `null`.

## The Challenge

Explain the difference between `null` and `undefined` in JavaScript. What are the key similarities and differences between them, and when would you use one over the other?


> **Common Mistake:** A junior engineer might think that they are the same thing. They might not be aware of the difference in their meaning or how they are used in different contexts.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `null` and `undefined`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Key Differences

| Feature      | `null`                                                              | `undefined`                                                              |
| ------------ | ------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Meaning**  | The intentional absence of any object value.                          | A variable that has been declared but has not yet been assigned a value. |
| **`typeof`** | `object` (this is a bug in JavaScript).                             | `undefined`.                                                            |
| **Usage**    | Used to indicate that a variable has no value.                      | Used by the JavaScript engine to indicate that a variable has not been initialized. |

### Step 2: Choose the Right Tool for the Job

In modern JavaScript, you should almost always use **`null`** to indicate that a variable has no value. You should almost never use **`undefined`**.

The only time you would use `undefined` is when you are checking if a variable has been initialized.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`null`:**

```javascript
let myVar = null;

if (myVar === null) {
  // ...
}
```

**`undefined`:**

```javascript
let myVar;

if (myVar === undefined) {
  // ...
}
```


---

### Quick Check

**What will `null == undefined` return?**

-> A. **`true`**
   B. `false`
   C. `TypeError`
   D. `SyntaxError`

<details>
<summary>See Answer</summary>

`null == undefined` returns `true`. This is because the loose equality operator `==` considers `null` and `undefined` to be equal.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

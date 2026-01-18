# Complete Node.js Interview Guide

Master Node.js with these real-world interview questions covering core concepts, asynchronous programming, and building scalable applications. Practice scenarios that mirror actual backend engineering challenges.

**Companies that ask these questions:** Netflix | LinkedIn | Uber | PayPal | Trello

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | Your Node.js service is unresponsive. How do you debug a blo... | Debugging | Core Concepts |
| 2 | What is the difference between callbacks, promises, and asyn... | Conceptual | Asynchronous Programming |
| 3 | How do you handle errors in asynchronous Node.js code? | Practical | Error Handling |
| 4 | What are streams and why are they useful in Node.js? | Conceptual | Core Concepts |
| 5 | How do you scale a Node.js application? | Architecture | Scalability |
| 6 | Your Node.js service has a memory leak. How do you debug it? | Debugging | Debugging |
| 7 | What is the difference between `require` and `import` in Nod... | Conceptual | Core Concepts |
| 8 | What is the difference between `child_process` and `worker_t... | Conceptual | Concurrency |
| 9 | How does Node.js handle I/O operations? | Conceptual | Core Concepts |
| 10 | What is the `Buffer` class and what is it used for in Node.j... | Conceptual | Core Concepts |
| 11 | What is the difference between `npm` and `yarn`? | Conceptual | Ecosystem |
| 12 | How do you secure a Node.js application? | Practical | Security |

---

## What You'll Learn

- Understand the core concepts of Node.js.
- Master asynchronous programming with callbacks, promises, and async/await.
- Build and deploy scalable microservices.
- Debug and optimize Node.js applications.
- Work with the Node.js ecosystem of modules and libraries.

---

## Interview Questions

### Question 1: Your Node.js service is unresponsive. How do you debug a blocked event loop?

**Type:** Debugging | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are responsible for a Node.js microservice that handles user authentication. The service is experiencing intermittent periods of unresponsiveness, where it stops responding to requests for several seconds at a time.

You have already checked the usual suspects: the database is healthy, the network is stable, and there are no obvious errors in the logs. You suspect that the event loop might be blocked.

## The Challenge

Explain what the Node.js event loop is and how it can become blocked. What are the common causes of a blocked event loop, and how would you debug this issue?


> **Common Mistake:** A junior engineer might not be aware of the event loop. They might try to debug the problem by adding `console.log` statements to the code, which would not be very effective. They might also not know how to use the built-in profiler or other debugging tools.

> **Senior Engineer Approach:** A senior engineer would immediately suspect that the event loop is blocked. They would be able to explain what the event loop is and how it can become blocked. They would also have a clear plan for how to debug the issue, including using the built-in profiler and other tools to identify the source of the problem.

### Step 1: Understand the Event Loop

The Node.js event loop is a single-threaded, non-blocking I/O mechanism that allows Node.js to handle a large number of concurrent connections efficiently. It works by offloading I/O operations to the operating system and then using a queue to process the results of those operations.

A blocked event loop occurs when a long-running synchronous operation prevents the event loop from processing other events in the queue. This can cause the entire application to become unresponsive.

### Step 2: Identify the Source of the Problem

Here are some common causes of a blocked event loop:

| Cause                     | Example                                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------- |
| **Long-running synchronous code** | A complex calculation, a large loop, or a synchronous file I/O operation.                         |
| **CPU-intensive operations**      | Image processing, video encoding, or cryptography.                                              |
| **Poorly written regex**      | A regular expression that takes a long time to execute.                                           |
| **JSON.parse and JSON.stringify** | Parsing or stringifying a large JSON object can be a blocking operation.                         |

### Step 3: Debug the Issue

Here's how you can debug a blocked event loop:

**1. Use the built-in profiler:**

Node.js has a built-in profiler that can be used to identify the parts of your code that are taking the most time to execute.

```bash
node --prof my_script.js
```

This will generate a file called `isolate-*.log` that can be analyzed with the `--prof-process` flag.

**2. Use a third-party tool:**

There are also several third-party tools that can be used to debug a blocked event loop, such as Clinic.js and `0x`.

**3. Use a heap dump:**

If you suspect that the issue is with a large JSON object, you can use a heap dump to inspect the memory usage of your application.

### Step 4: Fix the Problem

Once you have identified the source of the problem, you can fix it by:

-   **Breaking up long-running synchronous code** into smaller, asynchronous chunks.
-   **Offloading CPU-intensive operations** to a separate worker thread or a different service.
-   **Using a more efficient regex engine** or by optimizing your regular expressions.
-   **Using a streaming JSON parser** to parse large JSON objects.


---

### Quick Check

**Which of the following is the most likely cause of a blocked event loop?**

   A. A database query that takes a long time to execute.
-> B. **A long-running `for` loop.**
   C. A file I/O operation that takes a long time to complete.
   D. An HTTP request to an external API.

<details>
<summary>See Answer</summary>

While all of these can cause performance issues, a long-running `for` loop is the most likely cause of a blocked event loop, because it is a synchronous, CPU-intensive operation that will block the event loop until it is complete.

</details>

---

### Question 2: What is the difference between callbacks, promises, and async/await?

**Type:** Conceptual | **Category:** Asynchronous Programming

## The Scenario

You are a backend engineer at a fintech company. You are writing a new service that needs to make several asynchronous calls to other services to get the data it needs.

You are trying to decide whether to use callbacks, promises, or async/await to handle the asynchronous calls.

## The Challenge

Explain the difference between callbacks, promises, and async/await. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might only be familiar with one of these approaches. They might not be aware of the trade-offs between them, or they might not know how to choose the right one for a given use case.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between callbacks, promises, and async/await. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Evolution of Asynchronous Programming in Node.js

| Approach      | Description                                                                                             | Pros                                                              | Cons                                                               |
| ------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Callbacks** | A function that is passed as an argument to another function and is executed when the other function has completed. | Simple and easy to understand for basic use cases.                | Can lead to "callback hell" (nested callbacks that are difficult to read and maintain). |
| **Promises**  | An object that represents the eventual completion (or failure) of an asynchronous operation.             | More readable and easier to chain than callbacks.                  | Can be more verbose than async/await.                             |
| **Async/await** | Syntactic sugar on top of promises that makes asynchronous code look and behave more like synchronous code. | Very readable and easy to write.                                  | Can be more difficult to debug than promises if you are not careful. |

### Step 2: Choose the Right Tool for the Job

For our use case, `async/await` is the best choice. It is the most modern and readable way to write asynchronous code in Node.js, and it is well-suited for handling multiple asynchronous calls.

### Step 3: Code Examples

Here are some code examples that show the difference between the three approaches:

**Callbacks:**

```javascript
const fs = require('fs');

fs.readFile('file1.txt', 'utf8', (err, data1) => {
  if (err) throw err;
  fs.readFile('file2.txt', 'utf8', (err, data2) => {
    if (err) throw err;
    console.log(data1, data2);
  });
});
```

**Promises:**

```javascript
const fs = require('fs').promises;

fs.readFile('file1.txt', 'utf8')
  .then(data1 => {
    return fs.readFile('file2.txt', 'utf8')
      .then(data2 => {
        console.log(data1, data2);
      });
  })
  .catch(err => {
    throw err;
  });
```

**Async/await:**

```javascript
const fs = require('fs').promises;

async function readFiles() {
  try {
    const data1 = await fs.readFile('file1.txt', 'utf8');
    const data2 = await fs.readFile('file2.txt', 'utf8');
    console.log(data1, data2);
  } catch (err) {
    throw err;
  }
}

readFiles();
```

As you can see, `async/await` is the most readable and concise of the three approaches.


---

### Quick Check

**You need to make three asynchronous calls in a sequence, where each call depends on the result of the previous call. Which approach would be the most readable?**

   A. Callbacks
   B. Promises
-> C. **Async/await**
   D. All are equally readable

<details>
<summary>See Answer</summary>

Async/await is the most readable approach for this use case. It allows you to write asynchronous code that looks and behaves like synchronous code, which makes it much easier to read and understand.

</details>

---

### Question 3: How do you handle errors in asynchronous Node.js code?

**Type:** Practical | **Category:** Error Handling

## The Scenario

You are a backend engineer at an e-commerce company. You are writing a new service that makes several asynchronous calls to other services to process an order.

You need to make sure that you handle errors correctly at each step of the process. If any of the asynchronous calls fail, you need to be able to catch the error and respond to the user with an appropriate error message.

## The Challenge

Explain how you would handle errors in asynchronous Node.js code. What are the different error handling patterns that you would use for callbacks, promises, and async/await?


> **Common Mistake:** A junior engineer might not have a clear strategy for handling errors in asynchronous code. They might just wrap their code in a `try...catch` block, which will not work for all asynchronous patterns. They might also not be aware of the importance of using a centralized error handling middleware.

> **Senior Engineer Approach:** A senior engineer would have a deep understanding of the different error handling patterns for asynchronous code. They would be able to explain how to use error-first callbacks, `.catch()` for promises, and `try...catch` for async/await. They would also know how to use a centralized error handling middleware to handle all errors in a consistent way.

### Step 1: Choose the Right Pattern for Each Asynchronous Approach

| Approach      | Error Handling Pattern                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Callbacks** | Error-first callbacks: the first argument to the callback function is always the error.                               |
| **Promises**  | The `.catch()` method is used to handle errors.                                                                    |
| **Async/await** | The `try...catch` block is used to handle errors.                                                                  |

### Step 2: Code Examples

Here are some code examples that show how to handle errors in each of the three approaches:

**Callbacks:**

```javascript
const fs = require('fs');

fs.readFile('file1.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  // ...
});
```

**Promises:**

```javascript
const fs = require('fs').promises;

fs.readFile('file1.txt', 'utf8')
  .then(data => {
    // ...
  })
  .catch(err => {
    console.error(err);
  });
```

**Async/await:**

```javascript
const fs = require('fs').promises;

async function readFile() {
  try {
    const data = await fs.readFile('file1.txt', 'utf8');
    // ...
  } catch (err) {
    console.error(err);
  }
}
```

### Step 3: Use a Centralized Error Handling Middleware

In a real-world application, you would not want to handle errors in each individual route handler. Instead, you would use a centralized error handling middleware to handle all errors in a consistent way.

In an Express.js application, you can define an error handling middleware like this:

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

This middleware will be called whenever an error occurs in your application.


---

### Quick Check

**You are using `async/await` and you want to handle errors from multiple `await` calls in a single place. Which of the following would you use?**

   A. An error-first callback
   B. A `.catch()` block
-> C. **A `try...catch` block**
   D. A `finally` block

<details>
<summary>See Answer</summary>

A `try...catch` block is the correct choice for this task. You can wrap all your `await` calls in a single `try...catch` block to handle any errors that occur.

</details>

---

### Question 4: What are streams and why are they useful in Node.js?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a video streaming company. You are building a new service that will allow users to upload and stream large video files.

You need to find a way to handle these large files efficiently, without loading the entire file into memory at once.

## The Challenge

Explain what streams are in Node.js and why they are useful for this use case. What are the different types of streams, and how would you use them to build the video streaming service?


> **Common Mistake:** A junior engineer might try to solve this problem by reading the entire file into memory using `fs.readFileSync()`. This would be very inefficient and would likely cause the application to crash for large files.

> **Senior Engineer Approach:** A senior engineer would know that streams are the perfect tool for this job. They would be able to explain the different types of streams and would have a clear plan for how to use them to build an efficient and scalable video streaming service.

### Step 1: Understand What Streams Are

Streams are a way of reading and writing data in chunks, without having to load the entire file into memory at once. This makes them very efficient for handling large amounts of data.

### Step 2: The Different Types of Streams

There are four main types of streams in Node.js:

| Type          | Description                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| **Readable**  | A stream from which data can be read (e.g., `fs.createReadStream()`).                                   |
| **Writable**  | A stream to which data can be written (e.g., `fs.createWriteStream()`).                                  |
| **Duplex**    | A stream that is both readable and writable (e.g., a TCP socket).                                        |
| **Transform** | A stream that can be used to modify or transform the data as it is being read or written (e.g., a `zlib` stream). |

### Step 3: Build the Video Streaming Service

Here's how we can use streams to build the video streaming service:

**1. Create a readable stream for the video file:**

```javascript
const fs = require('fs');

const readStream = fs.createReadStream('my_video.mp4');
```

**2. Create a writable stream for the HTTP response:**

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // ...
});
```

**3. Use `pipe()` to connect the two streams:**

The `pipe()` method is a convenient way to connect a readable stream to a writable stream. It automatically handles the flow of data, so you don't have to worry about backpressure.

```javascript
readStream.pipe(res);
```

By using streams, we can efficiently stream the video file to the user without having to load the entire file into memory at once.


---

### Quick Check

**You are building a service that needs to compress a large file before sending it to the user. Which type of stream would you use?**

   A. Readable
   B. Writable
   C. Duplex
-> D. **Transform**

<details>
<summary>See Answer</summary>

A transform stream is the correct choice for this task. It allows you to modify or transform the data as it is being read or written. In this case, you would use a `zlib` stream to compress the data.

</details>

---

### Question 5: How do you scale a Node.js application?

**Type:** Architecture | **Category:** Scalability

## The Scenario

You are a backend engineer at a fast-growing startup. Your company's main application is a monolith that is written in Node.js. The application is starting to experience performance issues, and you have been tasked with coming up with a plan to scale it.

## The Challenge

Explain your strategy for scaling the Node.js application. What are the different scaling strategies that you would use, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might only be aware of vertical scaling (i.e., adding more resources to a single machine). They might not be aware of horizontal scaling or the other scaling strategies that are available.

> **Senior Engineer Approach:** A senior engineer would know that there are multiple ways to scale a Node.js application. They would be able to explain the difference between vertical scaling and horizontal scaling, and they would have a clear plan for how to use both strategies to scale the application.

### Step 1: Vertical Scaling

The first step is to scale the application vertically. This means adding more resources (e.g., CPU, memory) to the machine that is running the application.

| Pros                               | Cons                                                              |
| ---------------------------------- | ----------------------------------------------------------------- |
| Easy to implement.                 | Can be expensive.                                                 |
| Can provide a significant performance improvement. | There is a limit to how much you can scale a single machine. |

### Step 2: Horizontal Scaling

The next step is to scale the application horizontally. This means adding more machines to the cluster and distributing the load across them.

There are two main ways to do this in Node.js:

**1. The `cluster` module:**

The `cluster` module is a built-in module in Node.js that allows you to create a cluster of processes that share the same server port.

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);
}
```

**2. A process manager like PM2:**

PM2 is a popular process manager for Node.js that can be used to automatically scale your application across multiple processes.

### Step 3: Microservices

The final step is to break the monolith down into a set of smaller, independent microservices. This can provide a number of benefits, including:

-   **Improved scalability:** You can scale each microservice independently.
-   **Improved fault tolerance:** If one microservice fails, the others will not be affected.
-   **Improved maintainability:** It is easier to maintain a small, focused microservice than a large, complex monolith.

By using a combination of these scaling strategies, you can build a Node.js application that is scalable, reliable, and easy to maintain.


---

### Quick Check

**You are building a real-time chat application that needs to be able to handle a large number of concurrent connections. Which of the following scaling strategies would be the most appropriate?**

   A. Vertical scaling
   B. Horizontal scaling with the `cluster` module
   C. Horizontal scaling with microservices
-> D. **All of the above**

<details>
<summary>See Answer</summary>

All of these strategies can be used to scale a real-time chat application. However, a combination of horizontal scaling with microservices is likely to be the most effective approach. This will allow you to scale the different parts of the application independently and to improve the fault tolerance of the system.

</details>

---

### Question 6: Your Node.js service has a memory leak. How do you debug it?

**Type:** Debugging | **Category:** Debugging

## The Scenario

You are a backend engineer at a social media company. You are responsible for a Node.js microservice that is experiencing a memory leak. The service's memory usage is constantly increasing, and it eventually crashes.

You need to find the source of the memory leak and fix it.

## The Challenge

Explain your strategy for debugging a memory leak in a Node.js application. What are the common causes of memory leaks, and what tools would you use to identify the source of the leak?


> **Common Mistake:** A junior engineer might try to debug the problem by adding `console.log` statements to the code to track the memory usage. This would be very inefficient and would not provide much insight into the source of the leak.

> **Senior Engineer Approach:** A senior engineer would have a clear strategy for debugging a memory leak. They would be able to explain the common causes of memory leaks and would know how to use tools like the built-in profiler, heap dumps, and third-party tools to identify the source of the leak.

### Step 1: Understand the Common Causes of Memory Leaks

| Cause                     | Example                                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------- |
| **Global variables**      | Storing data in global variables that is never garbage collected.                                     |
| **Closures**              | Creating closures that hold references to large objects.                                            |
| **Event listeners**       | Adding event listeners that are never removed.                                                      |
| **Caches**                | Storing data in a cache that is never cleared.                                                      |

### Step 2: Use a Heap Dump to Identify the Source of the Leak

A heap dump is a snapshot of the memory usage of your application. You can use a heap dump to identify the objects that are taking up the most memory and to see what is holding a reference to them.

You can create a heap dump using the `heapdump` library or by using the built-in `--inspect` flag.

```bash
node --inspect my_script.js
```

Once you have the heap dump, you can use a tool like the Chrome DevTools to analyze it.

### Step 3: Use a Memory Profiler

A memory profiler can be used to track the memory usage of your application over time. This can be useful for identifying the parts of your code that are allocating the most memory.

You can use the built-in profiler in Node.js or a third-party tool like Clinic.js to profile the memory usage of your application.

### Step 4: Fix the Leak

Once you have identified the source of the leak, you can fix it by:

-   **Removing unnecessary references** to objects so that they can be garbage collected.
-   **Using a weak reference** to an object if you do not want to prevent it from being garbage collected.
-   **Clearing caches** periodically.
-   **Removing event listeners** when they are no longer needed.


---

### Quick Check

**You have a memory leak in your application, and you suspect that it is being caused by a closure that is holding a reference to a large object. Which of the following would be the best way to fix the leak?**

-> A. **Set the object to `null` when you are finished with it.**
   B. Use a weak reference to the object.
   C. Store the object in a global variable.
   D. None of the above

<details>
<summary>See Answer</summary>

Setting the object to `null` will remove the reference to it, which will allow it to be garbage collected. Using a weak reference would also work, but it is a more advanced technique that is not always necessary.

</details>

---

### Question 7: What is the difference between `require` and `import` in Node.js?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are working on a new Node.js project and need to decide whether to use CommonJS modules (`require`) or ES modules (`import`).

## The Challenge

Explain the difference between `require` and `import`. What are the pros and cons of each approach, and which one would you choose for a new project?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference between CommonJS and ES modules, or the implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `require` and `import`. They would also be able to explain the benefits of using ES modules and would have a clear recommendation for which one to use in a new project.

### Step 1: Understand the Two Module Systems

| Feature         | `require` (CommonJS)                                                              | `import` (ES Modules)                                                              |
| --------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Syntax**      | Dynamic, can be called anywhere in the code.                                      | Static, must be at the top level of a module.                                      |
| **Loading**     | Synchronous, loads modules at runtime.                                              | Asynchronous, loads modules before execution.                                      |
| **Tree shaking**| Not supported.                                                                      | Supported, allows bundlers to remove unused code.                                |
| **Compatibility**| Supported in all versions of Node.js.                                           | Supported in modern versions of Node.js.                                         |

### Step 2: Choose the Right Tool for the Job

For a new project, you should use **ES modules (`import`)**. They are the standard for JavaScript, and they have several benefits over CommonJS modules, such as tree shaking and better support for circular dependencies.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`require` (CommonJS):**

```javascript
// math.js
function add(a, b) {
  return a + b;
}
module.exports = { add };

// main.js
const { add } = require('./math.js');
console.log(add(2, 3));
```

**`import` (ES Modules):**

To use ES modules in Node.js, you need to either use the `.mjs` file extension or set `"type": "module"` in your `package.json` file.

```javascript
// math.mjs
export function add(a, b) {
  return a + b;
}

// main.mjs

console.log(add(2, 3));
```


---

### Quick Check

**You are building a library and you want to make sure that it can be used by both CommonJS and ES modules. What should you do?**

   A. Write the library in CommonJS.
   B. Write the library in ES modules.
-> C. **Write the library in both CommonJS and ES modules.**
   D. It's not possible to support both.

<details>
<summary>See Answer</summary>

To support both CommonJS and ES modules, you should write the library in ES modules and then use a tool like Babel to transpile it to CommonJS. You can then specify both the ES module and the CommonJS entry points in your `package.json` file.

</details>

---

### Question 8: What is the difference between `child_process` and `worker_threads`?

**Type:** Conceptual | **Category:** Concurrency

## The Scenario

You are a backend engineer at a data processing company. You are building a new service that needs to perform a CPU-intensive operation, such as image processing or video encoding.

You know that you should not perform this operation on the main thread, because it will block the event loop. You are considering using either the `child_process` module or the `worker_threads` module to offload the operation to a separate thread.

## The Challenge

Explain the difference between the `child_process` module and the `worker_threads` module. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might not be aware of the difference between these two modules. They might just choose one at random, without considering the trade-offs between them.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `child_process` and `worker_threads`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: `child_process` vs. `worker_threads`

| Feature           | `child_process`                                                              | `worker_threads`                                                                |
| ----------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Process Model** | Creates a new process.                                                       | Creates a new thread within the same process.                                   |
| **Memory**        | Each process has its own memory space.                                       | Threads share the same memory space.                                            |
| **Communication** | Communication between processes is done using IPC (inter-process communication). | Communication between threads is done using message passing or shared memory. |
| **Use Cases**     | Running external commands, creating long-running background processes.        | Performing CPU-intensive operations without blocking the event loop.          |

### Step 2: Choose the Right Tool for the Job

For our use case, **`worker_threads` is the best choice**. It is more lightweight than `child_process`, and it allows us to share memory between the main thread and the worker thread, which can be more efficient.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`child_process`:**

```javascript
// main.js
const { fork } = require('child_process');

const child = fork('worker.js');

child.on('message', (message) => {
  console.log('Message from child:', message);
});

child.send({ hello: 'world' });

// worker.js
process.on('message', (message) => {
  console.log('Message from parent:', message);
});

process.send({ foo: 'bar' });
```

**`worker_threads`:**

```javascript
// main.js
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  worker.on('message', (message) => {
    console.log('Message from worker:', message);
  });
  worker.postMessage({ hello: 'world' });
} else {
  parentPort.on('message', (message) => {
    console.log('Message from parent:', message);
  });
  parentPort.postMessage({ foo: 'bar' });
}
```


---

### Quick Check

**You need to run an external command, such as `git`, from your Node.js application. Which of the following would you use?**

-> A. **`child_process`**
   B. `worker_threads`
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

`child_process` is the correct choice for this task. It is designed for running external commands and creating long-running background processes.

</details>

---

### Question 9: How does Node.js handle I/O operations?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to handle a large number of concurrent I/O operations, such as reading from a database, writing to a file, and making HTTP requests to other services.

You need to understand how Node.js handles I/O operations so that you can build a service that is both performant and scalable.

## The Challenge

Explain how Node.js handles I/O operations. What is the role of the event loop, and how does it allow Node.js to handle a large number of concurrent connections with a single thread?


> **Common Mistake:** A junior engineer might think that Node.js uses multiple threads to handle I/O operations. They might not be aware of the event loop or the concept of non-blocking I/O.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of how Node.js handles I/O operations. They would be able to explain the role of the event loop, libuv, and the thread pool. They would also be able to explain the benefits of using a non-blocking I/O model.

### Step 1: The Event Loop

The Node.js event loop is a single-threaded, non-blocking I/O mechanism that is the key to Node.js's ability to handle a large number of concurrent connections.

### Step 2: Non-Blocking I/O

When an I/O operation is performed in Node.js, it is offloaded to the operating system. The event loop does not wait for the operation to complete. Instead, it continues to process other events in the queue.

When the I/O operation is complete, the operating system notifies the event loop, and the event loop then executes the corresponding callback function.

### Step 3: `libuv` and the Thread Pool

Node.js uses a C++ library called `libuv` to handle the low-level details of the event loop and the thread pool. The thread pool is a pool of worker threads that can be used to perform CPU-intensive operations without blocking the event loop.

| Component      | Description                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------- |
| **Event Loop** | A single thread that is responsible for processing events in the queue.                                |
| **`libuv`**    | A C++ library that provides the low-level implementation of the event loop and the thread pool.           |
| **Thread Pool**| A pool of worker threads that can be used to perform CPU-intensive operations.                          |

### Step 4: The Benefits of Non-Blocking I/O

The non-blocking I/O model used by Node.js has several benefits:

-   **Scalability:** It allows Node.js to handle a large number of concurrent connections with a single thread.
-   **Performance:** It can be very performant, especially for I/O-bound applications.
-   **Simplicity:** It simplifies the process of writing asynchronous code.


---

### Quick Check

**You are building a service that needs to perform a CPU-intensive operation, such as image processing. Which of the following would be the best way to do this without blocking the event loop?**

   A. Use a `for` loop.
   B. Use a `while` loop.
-> C. **Offload the operation to a worker thread using the `worker_threads` module.**
   D. Perform the operation on the main thread.

<details>
<summary>See Answer</summary>

Offloading the operation to a worker thread is the correct choice for this task. This will allow the event loop to continue processing other events while the CPU-intensive operation is being performed.

</details>

---

### Question 10: What is the `Buffer` class and what is it used for in Node.js?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to interact with a TCP server that sends and receives binary data.

You need to find a way to handle this binary data in your Node.js application.

## The Challenge

Explain what the `Buffer` class is in Node.js and why it is useful for this use case. What are the key methods of the `Buffer` class, and how would you use them to read and write binary data?


> **Common Mistake:** A junior engineer might try to handle the binary data as a string. This would not work, because strings in JavaScript are not designed to handle binary data. They might not be aware of the `Buffer` class, which is the correct tool for this job.

> **Senior Engineer Approach:** A senior engineer would know that the `Buffer` class is the key to handling binary data in Node.js. They would be able to explain what a `Buffer` is and how to use it to read and write binary data.

### Step 1: Understand What a `Buffer` Is

A `Buffer` is a global class in Node.js that is used to handle binary data. It is similar to an array of integers, but it corresponds to a raw memory allocation outside the V8 heap.

### Step 2: The Key Methods of the `Buffer` Class

| Method          | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **`Buffer.from()`** | Creates a new `Buffer` from a string, array, or another `Buffer`.                                     |
| **`buf.write()`** | Writes a string to a `Buffer`.                                                                        |
| **`buf.toString()`**| Decodes a `Buffer` into a string.                                                                       |
| **`buf.readInt8()`**| Reads a signed 8-bit integer from a `Buffer`.                                                          |
| **`buf.writeInt8()`**| Writes a signed 8-bit integer to a `Buffer`.                                                           |

### Step 3: Read and Write Binary Data

Here's how we can use the `Buffer` class to read and write binary data from a TCP socket:

```javascript
const net = require('net');

const client = net.createConnection({ port: 8124 }, () => {
  // Create a buffer with a single byte
  const buf = Buffer.from([0x1]);

  // Write the buffer to the socket
  client.write(buf);
});

client.on('data', (data) => {
  // Read the first byte from the buffer
  const byte = data.readInt8(0);

  console.log('Received:', byte);
});
```

By using the `Buffer` class, we can easily read and write binary data in our Node.js application.

---

### Question 11: What is the difference between `npm` and `yarn`?

**Type:** Conceptual | **Category:** Ecosystem

## The Scenario

You are starting a new Node.js project and need to decide whether to use `npm` or `yarn` as your package manager.

## The Challenge

Explain the difference between `npm` and `yarn`. What are the pros and cons of each approach, and which one would you choose for a new project?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the historical context of why `yarn` was created, or the differences in performance and features between the two.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `npm` and `yarn`. They would also be able to explain the benefits of using `yarn` and would have a clear recommendation for which one to use in a new project.

### Step 1: Understand the History

`npm` is the default package manager for Node.js. It was created in 2010 and has been the standard for many years.

`yarn` was created by Facebook in 2016 to address some of the shortcomings of `npm` at the time, such as performance and security.

### Step 2: `npm` vs. `yarn`

| Feature              | `npm`                                                                                             | `yarn`                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Performance**      | Slower than `yarn` in older versions, but has improved significantly in recent versions.            | Faster than `npm` in older versions, due to parallel package installation and caching.              |
| **Lock file**        | `package-lock.json`                                                                               | `yarn.lock`                                                                                       |
| **Workspaces**       | Supported in `npm` 7 and later.                                                                   | Supported in `yarn` since version 1.                                                              |
| **Plug'n'Play (PnP)** | Not supported.                                                                                    | A feature that allows you to run your project without a `node_modules` folder.                    |

### Step 3: Choose the Right Tool for the Job

For a new project, **`yarn` is generally the better choice**. It is faster, more secure, and has more features than `npm`.

However, `npm` has improved significantly in recent years, and the performance difference between the two is not as great as it used to be. If you are working on a project that already uses `npm`, there is no need to switch to `yarn`.


---

### Quick Check

**You are working on a large monorepo with many packages. Which feature of `yarn` would be the most helpful?**

   A. Performance
   B. Lock file
-> C. **Workspaces**
   D. Plug'n'Play

<details>
<summary>See Answer</summary>

Workspaces are the most helpful feature of `yarn` for working with monorepos. They allow you to manage the dependencies of all your packages in a single place, which can save you a lot of time and effort.

</details>

---

### Question 12: How do you secure a Node.js application?

**Type:** Practical | **Category:** Security

## The Scenario

You are a backend engineer at a fintech company. You are responsible for a Node.js microservice that handles user authentication and authorization.

You need to make sure that the service is secure and that it is protected against common vulnerabilities, such as cross-site scripting (XSS), SQL injection, and cross-site request forgery (CSRF).

## The Challenge

Explain your strategy for securing the Node.js application. What are the key security best practices that you would follow, and what tools would you use to help you?


> **Common Mistake:** A junior engineer might not have a clear strategy for securing a Node.js application. They might not be aware of common vulnerabilities or the tools that can be used to protect against them.

> **Senior Engineer Approach:** A senior engineer would have a deep understanding of security best practices. They would be able to explain how to protect against common vulnerabilities, and they would have a clear plan for how to use tools like `helmet`, `csurf`, and `express-validator` to secure the application.

### Step 1: Use a Security Linter

The first step is to use a security linter, such as `eslint-plugin-security`, to automatically detect security vulnerabilities in your code.

### Step 2: Protect Against Common Vulnerabilities

Here are some common vulnerabilities and how to protect against them:

| Vulnerability | How to protect against it                                                                                             |
| ------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **XSS**       | Use a library like `helmet` to set security-related HTTP headers. Sanitize user input to escape any malicious code.     |
| **SQL Injection** | Use an ORM like Sequelize or TypeORM to automatically escape SQL queries.                                            |
| **CSRF**      | Use a library like `csurf` to generate and validate CSRF tokens.                                                       |
| **Insecure Deserialization** | Avoid using `eval()` and other functions that can execute arbitrary code. Use a safe serialization format like JSON. |

### Step 3: Use Security-Related HTTP Headers

You can use a library like `helmet` to set security-related HTTP headers, such as `Content-Security-Policy`, `Strict-Transport-Security`, and `X-Frame-Options`.

```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();
app.use(helmet());
```

### Step 4: Validate User Input

You should always validate user input to make sure that it is in the correct format and that it does not contain any malicious code. You can use a library like `express-validator` to do this.

```javascript
const { body, validationResult } = require('express-validator');

app.post('/user', body('email').isEmail(), (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // ...
});
```

### Step 5: Keep Your Dependencies Up-to-Date

You should regularly update your dependencies to make sure that you are not using any versions with known security vulnerabilities. You can use a tool like `npm audit` to check for vulnerabilities in your dependencies.


---

### Quick Check

**You are building a new REST API and want to protect it against CSRF attacks. Which of the following would be the most effective?**

   A. Use HTTPS.
   B. Use a firewall.
-> C. **Use CSRF tokens.**
   D. Use a rate limiter.

<details>
<summary>See Answer</summary>

CSRF tokens are the most effective way to protect against CSRF attacks. A CSRF token is a unique, secret, and unpredictable value that is generated by the server and sent to the client. The client then includes the token in all subsequent requests, and the server validates the token before processing the request.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

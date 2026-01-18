# Complete TypeScript Interview Guide

Master TypeScript with these real-world interview questions covering core concepts, generics, and advanced types. Practice scenarios that mirror actual frontend and backend engineering challenges.

**Companies that ask these questions:** Microsoft | Google | Facebook | Netflix | Amazon

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What is the difference between an `interface` and a `type` i... | Conceptual | Core Concepts |
| 2 | What are generics and how do you use them in TypeScript? | Conceptual | Generics |
| 3 | What are decorators and how do you use them in TypeScript? | Conceptual | Decorators |
| 4 | What are the different access modifiers in TypeScript? | Conceptual | Object-Oriented Programming |
| 5 | What is the difference between `any` and `unknown` in TypeSc... | Conceptual | Type System |
| 6 | What are utility types and how do you use them in TypeScript... | Conceptual | Advanced Types |
| 7 | What are mapped types and how do you use them in TypeScript? | Conceptual | Advanced Types |
| 8 | What are conditional types and how do you use them in TypeSc... | Conceptual | Advanced Types |
| 9 | What are enums and how do you use them in TypeScript? | Conceptual | Core Concepts |
| 10 | What is the difference between a `namespace` and a `module` ... | Conceptual | Core Concepts |
| 11 | What are type guards and how do you use them in TypeScript? | Conceptual | Advanced Types |
| 12 | What are the `keyof` and `typeof` operators and how do you u... | Conceptual | Advanced Types |

---

## What You'll Learn

- Understand the core concepts of TypeScript.
- Master the type system.
- Write object-oriented and functional code.
- Debug and optimize TypeScript applications.
- Work with advanced features like generics and decorators.

---

## Interview Questions

### Question 1: What is the difference between an `interface` and a `type` in TypeScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to define the shape of an object.

You are not sure whether to use an `interface` or a `type` to define the shape of the object.

## The Challenge

Explain the difference between an `interface` and a `type` in TypeScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in features or the design implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between an `interface` and a `type`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Key Differences

| Feature      | `interface`                                                              | `type`                                                              |
| ------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Extensibility**| An `interface` can be extended by another `interface`.                | A `type` can be extended using an intersection.                     |
| **Declaration Merging** | An `interface` can be defined multiple times and the definitions will be merged. | A `type` cannot be defined multiple times.                         |
| **Use Cases**  | When you need to define the shape of an object.                         | When you need to define a union, a tuple, or a primitive type alias. |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use an **`interface`**. This is because we are defining the shape of an object, and an `interface` is the best tool for this job.

In general, you should use an `interface` when you are defining the shape of an object, and a `type` when you are defining a union, a tuple, or a primitive type alias.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`interface`:**

```typescript
interface MyInterface {
  myProperty: string;
}

const myObject: MyInterface = {
  myProperty: 'myValue'
};
```

**`type`:**

```typescript
type MyType = {
  myProperty: string;
};

const myObject: MyType = {
  myProperty: 'myValue'
};
```


---

### Quick Check

**You want to create a new type that can be either a `string` or a `number`. Which of the following would you use?**

   A. An `interface`
-> B. **A `type`**
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

A `type` is the correct choice for this task. It allows you to create a union type, which is a type that can be one of several other types.

</details>

---

### Question 2: What are generics and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Generics

## The Scenario

You are a frontend engineer at a fintech company. You are writing a new function that needs to work with a variety of different data types.

You could write a separate function for each data type, but this would be repetitive and would violate the DRY (Don't Repeat Yourself) principle.

## The Challenge

Explain what generics are in TypeScript and how you would use them to solve this problem. What are the key benefits of using generics?


> **Common Mistake:** A junior engineer might not be aware of generics. They might try to solve this problem by using the `any` type, which would not be type-safe.

> **Senior Engineer Approach:** A senior engineer would know that generics are the perfect tool for this job. They would be able to explain what generics are and how to use them to write type-safe code that can work with a variety of different data types.

### Step 1: Understand What Generics Are

Generics are a feature of TypeScript that allows you to write code that is type-safe and can work with a variety of different data types.

### Step 2: Write a Simple Generic Function

Here's how we can write a simple generic function that takes a value of any type and returns it:

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

In this example, `T` is a type parameter that can be replaced with any type.

### Step 3: Use the Generic Function

We can use the generic function with any type:

```typescript
let output1 = identity<string>("myString");
let output2 = identity<number>(123);
```

### The Benefits of Using Generics

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Type Safety**   | Generics provide compile-time type safety, which can help you to catch errors early.                    |
| **Reusability**   | You can reuse the same generic class or method with a variety of different data types.                  |
| **Readability**   | Generics make your code more readable by making it clear what types of data are being used.             |

### Generic Constraints

You can use a generic constraint to limit the types that can be used with a generic function or class.

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}
```

In this example, the `loggingIdentity` function will only accept types that have a `length` property.

---

### Question 3: What are decorators and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Decorators

## The Scenario

You are a frontend engineer at a fintech company. You are building a new framework that will be used to build REST APIs.

You want to be able to add metadata to your code, such as the HTTP method and the URL path, so that you can automatically generate the API documentation.

## The Challenge

Explain what decorators are in TypeScript and how you would use them to solve this problem. What are the key benefits of using decorators?


> **Common Mistake:** A junior engineer might not be aware of decorators. They might try to solve this problem by using comments or a separate configuration file, which would be a less elegant and less robust solution.

> **Senior Engineer Approach:** A senior engineer would know that decorators are the perfect tool for this job. They would be able to explain what decorators are and how to use them to add metadata to code. They would also have a clear plan for how to use decorators to automatically generate the API documentation.

### Step 1: Understand What Decorators Are

A decorator is a special kind of declaration that can be attached to a class declaration, method, accessor, property, or parameter. Decorators use the form `@expression`, where `expression` must evaluate to a function that will be called at runtime with information about the decorated declaration.

### Step 2: Write a Simple Decorator

Here's how we can write a simple decorator to specify the HTTP method and the URL path of a REST API endpoint:

```typescript
function RequestMapping(value: string, method: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // ... (store the metadata) ...
  };
}
```

### Step 3: Use the Decorator

We can use the decorator on a method in our controller class:

```typescript
class MyController {
  @RequestMapping("/hello", "GET")
  hello() {
    return "Hello, world!";
  }
}
```

### Step 4: Process the Decorator

We can use reflection to process the decorator at runtime and automatically generate the API documentation.

### The Benefits of Using Decorators

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Metadata**      | Decorators provide a way to add metadata to code, which can be used by tools and frameworks.         |
| **Readability**   | Decorators can make your code more readable by making it clear what the code is intended to do.         |
| **Extensibility** | You can create your own custom decorators to add new functionality to your code.                       |


---

### Quick Check

**You want to create a new decorator that can be used on both methods and properties. Which of the following would you use?**

   A. A method decorator
   B. A property decorator
-> C. **A decorator factory that returns a method or property decorator**
   D. None of the above

<details>
<summary>See Answer</summary>

A decorator factory is the correct choice for this task. It is a function that returns a decorator, and it can be used to create a decorator that can be used on multiple types of declarations.

</details>

---

### Question 4: What are the different access modifiers in TypeScript?

**Type:** Conceptual | **Category:** Object-Oriented Programming

## The Scenario

You are a frontend engineer at a fintech company. You are writing a new class that needs to have some private properties that can only be accessed by the class itself.

## The Challenge

Explain the different access modifiers in TypeScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might not be aware of access modifiers. They might just make all the properties of a class public, which would not be a very good solution.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the different access modifiers in TypeScript. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Modifier      | Description                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| **`public`**  | The default access modifier. A `public` property can be accessed from anywhere.                         |
| **`private`** | A `private` property can only be accessed from within the class that it is defined in.                  |
| **`protected`**| A `protected` property can be accessed from within the class that it is defined in and from any subclasses that extend it. |
| **`readonly`**| A `readonly` property can only be set in the constructor.                                               |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use the **`private`** access modifier. This will ensure that the properties can only be accessed from within the class.

### Step 3: Code Examples

Here are some code examples that show the difference between the different approaches:

```typescript
class MyClass {
  public myPublicProperty: string;
  private myPrivateProperty: string;
  protected myProtectedProperty: string;
  readonly myReadonlyProperty: string;

  constructor() {
    this.myPublicProperty = "hello";
    this.myPrivateProperty = "world";
    this.myProtectedProperty = "foo";
    this.myReadonlyProperty = "bar";
  }
}

class MySubclass extends MyClass {
  myMethod() {
    console.log(this.myPublicProperty); // OK
    // console.log(this.myPrivateProperty); // Error
    console.log(this.myProtectedProperty); // OK
    console.log(this.myReadonlyProperty); // OK
  }
}
```


---

### Quick Check

**You want to create a property that can be accessed by subclasses but not by the public. Which of the following would you use?**

   A. `public`
   B. `private`
-> C. **`protected`**
   D. `readonly`

<details>
<summary>See Answer</summary>

`protected` is the correct choice for this task. It allows a property to be accessed from within the class that it is defined in and from any subclasses that extend it.

</details>

---

### Question 5: What is the difference between `any` and `unknown` in TypeScript?

**Type:** Conceptual | **Category:** Type System

## The Scenario

You are a frontend engineer at a social media company. You are writing a new function that needs to work with a value of an unknown type.

You are not sure whether to use the `any` type or the `unknown` type.

## The Challenge

Explain the difference between the `any` and `unknown` types in TypeScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in type safety between the two.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `any` and `unknown`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | `any`                                                              | `unknown`                                                              |
| ------------ | ------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Type Safety**| Not type-safe. You can perform any operation on a value of type `any`. | Type-safe. You must perform a type check before you can perform any operation on a value of type `unknown`. |
| **Use Cases**  | When you need to opt-out of type checking for a particular value.     | When you have a value of an unknown type and you want to be type-safe. |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use the **`unknown`** type. This is because we have a value of an unknown type, and we want to be type-safe.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`any`:**

```typescript
let myVar: any;

myVar.myMethod(); // No error at compile time, but will fail at runtime
```

**`unknown`:**

```typescript
let myVar: unknown;

// This will raise an error at compile time
// myVar.myMethod();

if (typeof myVar === 'object' && myVar !== null && 'myMethod' in myVar) {
  // We can now call myMethod()
}
```


---

### Quick Check

**You are working with a value that you know is a string, but you want to be able to call any method on it without getting a type error. Which of the following would you use?**

-> A. **`any`**
   B. `unknown`
   C. `string`
   D. None of the above

<details>
<summary>See Answer</summary>

`any` is the correct choice for this task. It allows you to opt-out of type checking for a particular value, which can be useful for debugging or working with legacy code.

</details>

---

### Question 6: What are utility types and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Advanced Types

## The Scenario

You are a frontend engineer at a social media company. You are writing a new function that needs to take a subset of the properties of an existing type.

You could create a new type with only the properties that you need, but this would be repetitive and would violate the DRY (Don't Repeat Yourself) principle.

## The Challenge

Explain what utility types are in TypeScript and how you would use them to solve this problem. What are the key benefits of using utility types?


> **Common Mistake:** A junior engineer might not be aware of utility types. They might try to solve this problem by creating a new type with only the properties that they need, which would be a verbose and difficult to maintain solution.

> **Senior Engineer Approach:** A senior engineer would know that utility types are the perfect tool for this job. They would be able to explain what utility types are and how to use them to create new types from existing types.

### Step 1: Understand What Utility Types Are

Utility types are a set of built-in types in TypeScript that allow you to create new types from existing types.

### Step 2: The Key Utility Types

| Type          | Description                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| **`Partial<T>`** | Creates a new type with all the properties of `T` set to optional.                                    |
| **`Readonly<T>`**| Creates a new type with all the properties of `T` set to `readonly`.                                  |
| **`Pick<T, K>`** | Creates a new type by picking a set of properties `K` from `T`.                                         |
| **`Omit<T, K>`** | Creates a new type by omitting a set of properties `K` from `T`.                                        |
| **`Record<K, T>`**| Creates a new type with a set of properties `K` of type `T`.                                          |

### Step 3: Solve the Problem

For our use case, we can use the **`Pick`** utility type to create a new type with only the properties that we need.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type UserSummary = Pick<User, "id" | "name">;

const userSummary: UserSummary = {
  id: 1,
  name: "John Smith"
};
```


---

### Quick Check

**You want to create a new type that has all the properties of an existing type, but with all the properties set to optional. Which of the following would you use?**

-> A. **`Partial<T>`**
   B. `Readonly<T>`
   C. `Pick<T, K>`
   D. `Omit<T, K>`

<details>
<summary>See Answer</summary>

`Partial<T>` is the correct choice for this task. It creates a new type with all the properties of `T` set to optional.

</details>

---

### Question 7: What are mapped types and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Advanced Types

## The Scenario

You are a frontend engineer at a social media company. You are writing a new function that needs to take an object and return a new object with all the properties of the original object set to `readonly`.

You could create a new type with all the properties set to `readonly` by hand, but this would be repetitive and would violate the DRY (Don't Repeat Yourself) principle.

## The Challenge

Explain what mapped types are in TypeScript and how you would use them to solve this problem. What are the key benefits of using mapped types?


> **Common Mistake:** A junior engineer might not be aware of mapped types. They might try to solve this problem by creating a new type with all the properties set to `readonly` by hand, which would be a verbose and difficult to maintain solution.

> **Senior Engineer Approach:** A senior engineer would know that mapped types are the perfect tool for this job. They would be able to explain what mapped types are and how to use them to create new types from existing types.

### Step 1: Understand What Mapped Types Are

A mapped type is a generic type which uses a union of `PropertyKey`s (frequently created by a `keyof`) to iterate through keys to create a type.

### Step 2: Write a Simple Mapped Type

Here's how we can write a simple mapped type to create a new type with all the properties of an existing type set to `readonly`:

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

This is the actual implementation of the built-in `Readonly<T>` utility type.

### Step 3: Use the Mapped Type

We can use the mapped type with any type:

```typescript
interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = {
  id: 1,
  name: "John Smith"
};

// This will raise an error
// user.id = 2;
```

### The Benefits of Using Mapped Types

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Reusability**   | You can reuse the same mapped type with a variety of different types.                                   |
| **Maintainability**| You can change the properties of all the types that use a mapped type by just changing the mapped type. |


---

### Quick Check

**You want to create a new type that has all the properties of an existing type, but with all the properties set to optional. Which of the following would you use?**

-> A. **A mapped type**
   B. A utility type
   C. An intersection type
   D. A union type

<details>
<summary>See Answer</summary>

A mapped type is the correct choice for this task. You can use a mapped type to create a new type with all the properties of an existing type set to optional. This is how the built-in `Partial<T>` utility type is implemented.

</details>

---

### Question 8: What are conditional types and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Advanced Types

## The Scenario

You are a frontend engineer at a social media company. You are writing a new function that needs to have a different return type depending on the type of its input.

You could write a separate function for each input type, but this would be repetitive and would violate the DRY (Don't Repeat Yourself) principle.

## The Challenge

Explain what conditional types are in TypeScript and how you would use them to solve this problem. What are the key benefits of using conditional types?


> **Common Mistake:** A junior engineer might not be aware of conditional types. They might try to solve this problem by using a series of `if` statements, which would not be a very elegant or type-safe solution.

> **Senior Engineer Approach:** A senior engineer would know that conditional types are the perfect tool for this job. They would be able to explain what conditional types are and how to use them to create a function with a dynamic return type.

### Step 1: Understand What Conditional Types Are

A conditional type is a type that is selected based on a condition. It has the following syntax:

```typescript
T extends U ? X : Y
```

If `T` extends `U`, the type is `X`. Otherwise, the type is `Y`.

### Step 2: Write a Simple Conditional Type

Here's how we can write a simple conditional type to get the type of the elements of an array:

```typescript
type ElementType<T> = T extends (infer U)[] ? U : T;
```

In this example, if `T` is an array type, the type is `U` (the type of the elements of the array). Otherwise, the type is `T`.

### Step 3: Solve the Problem

Here's how we can use a conditional type to create a function with a dynamic return type:

```typescript
function myFunc<T>(arg: T): ElementType<T> {
  // ...
}
```

Now, the return type of `myFunc` will depend on the type of its input.

### The `infer` Keyword

The `infer` keyword is used in conditional types to infer a type from another type. In the `ElementType` example above, we use `infer U` to infer the type of the elements of the array.


---

### Quick Check

**You want to create a new type that is the return type of a function. Which of the following would you use?**

   A. A conditional type
   B. A mapped type
-> C. **The `ReturnType` utility type**
   D. None of the above

<details>
<summary>See Answer</summary>

The `ReturnType` utility type is the correct choice for this task. It is a built-in utility type that uses a conditional type to get the return type of a function.

</details>

---

### Question 9: What are enums and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are writing a new feature that needs to represent a set of related constants, such as the different roles that a user can have.

You could use a set of `const` variables, but this would not be a very elegant solution.

## The Challenge

Explain what enums are in TypeScript and how you would use them to solve this problem. What are the key benefits of using enums?


> **Common Mistake:** A junior engineer might not be aware of enums. They might try to solve this problem by using a set of `const` variables, which would be a less elegant and less type-safe solution.

> **Senior Engineer Approach:** A senior engineer would know that enums are the perfect tool for this job. They would be able to explain what enums are and how to use them to create a set of named constants.

### Step 1: Understand What Enums Are

An enum is a way to define a set of named constants.

### Step 2: The Different Types of Enums

| Type        | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **Numeric** | The default type of enum. The first member is initialized to 0, and each subsequent member is incremented by 1. |
| **String**  | Each member must be initialized with a string value.                                                    |
| **Heterogeneous**| A mix of numeric and string members.                                                                    |

### Step 3: Use an Enum

Here's how we can use an enum to represent the different roles that a user can have:

```typescript
enum Role {
  Admin,
  Editor,
  Viewer
}

const myRole: Role = Role.Admin;
```

### The Benefits of Using Enums

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Readability**   | Enums make your code more readable by giving names to a set of related constants.                       |
| **Type Safety**   | Enums provide compile-time type safety, which can help you to catch errors early.                    |

---

### Question 10: What is the difference between a `namespace` and a `module` in TypeScript?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are working on a large application and need to find a way to organize your code.

You are not sure whether to use a `namespace` or a `module`.

## The Challenge

Explain the difference between a `namespace` and a `module` in TypeScript. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in scope or the design implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between a `namespace` and a `module`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | `namespace`                                                              | `module`                                                              |
| ------------ | ----------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Scope**    | Creates a new scope for a set of related objects.                       | A file with a top-level `import` or `export` statement.               |
| **`export`** | Objects in a `namespace` are exported by default.                       | Objects in a `module` are not exported by default.                    |
| **Use Cases**  | For organizing code in a small application or a library.                | For organizing code in a large application.                           |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use **modules**. This is because we are working on a large application, and modules provide a more scalable and maintainable way to organize our code.

In general, you should use modules for all new projects. Namespaces are a legacy feature that should only be used for organizing code in old projects.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`namespace`:**

```typescript
namespace MyNamespace {
  export const myVar = "hello";
}

console.log(MyNamespace.myVar); // hello
```

**`module`:**

```typescript
// my-module.ts
export const myVar = "hello";

// main.ts


console.log(myVar); // hello
```


---

### Quick Check

**You are working on a large application and you want to organize your code into logical blocks of functionality. Which of the following would you use?**

   A. A `namespace`
-> B. **A `module`**
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

A `module` is the correct choice for this task. It allows you to organize your code into separate files, which makes it easier to maintain a large application.

</details>

---

### Question 11: What are type guards and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Advanced Types

## The Scenario

You are a frontend engineer at a social media company. You are writing a new function that takes a value of an unknown type and needs to perform a different operation depending on the type of the value.

You could use a series of `if` statements with `typeof` checks, but this would not be very elegant or type-safe.

## The Challenge

Explain what type guards are in TypeScript and how you would use them to solve this problem. What are the key benefits of using type guards?


> **Common Mistake:** A junior engineer might not be aware of type guards. They might try to solve this problem by using a series of `if` statements with `typeof` checks, which would be a verbose and error-prone solution.

> **Senior Engineer Approach:** A senior engineer would know that type guards are the perfect tool for this job. They would be able to explain what type guards are and how to use them to create a function with a dynamic return type.

### Step 1: Understand What Type Guards Are

A type guard is a function that returns a boolean value and that has a type predicate as its return type. A type predicate is a special return type that tells the TypeScript compiler that a value has a specific type if the function returns `true`.

### Step 2: Write a Simple Type Guard

Here's how we can write a simple type guard to check if a value is a `string`:

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}
```

### Step 3: Use the Type Guard

We can use the type guard in an `if` statement to narrow the type of a value:

```typescript
function myFunc(myVar: unknown) {
  if (isString(myVar)) {
    // myVar is now of type string
    console.log(myVar.toUpperCase());
  }
}
```

### The Benefits of Using Type Guards

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Type Safety**   | Type guards provide compile-time type safety, which can help you to catch errors early.                 |
| **Readability**   | Type guards make your code more readable by making it clear what type of data you are working with.    |
| **Reusability**   | You can reuse the same type guard in multiple places in your code.                                      |

---

### Question 12: What are the `keyof` and `typeof` operators and how do you use them in TypeScript?

**Type:** Conceptual | **Category:** Advanced Types

## The Scenario

You are a frontend engineer at a social media company. You are writing a new function that needs to take an object and a key of that object, and return the value of that key.

You want to make sure that the function is type-safe and that you can only pass valid keys of the object to the function.

## The Challenge

Explain what the `keyof` and `typeof` operators are in TypeScript and how you would use them to solve this problem.


> **Common Mistake:** A junior engineer might not be aware of the `keyof` and `typeof` operators. They might try to solve this problem by using a string to represent the key, which would not be type-safe.

> **Senior Engineer Approach:** A senior engineer would know that the `keyof` and `typeof` operators are the perfect tools for this job. They would be able to explain what these operators are and how to use them to create a type-safe function that can get a property of an object.

### Step 1: Understand What the `keyof` and `typeof` Operators Are

| Operator   | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **`keyof`**  | An operator that takes an object type and returns a string or numeric literal union of its keys.        |
| **`typeof`** | An operator that takes a value and returns its type.                                                    |

### Step 2: Solve the Problem

Here's how we can use the `keyof` and `typeof` operators to create a type-safe function that can get a property of an object:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const myObject = {
  a: 1,
  b: 2,
  c: 3
};

const myValue = getProperty(myObject, "a"); // 1
```

In this example, we use the `keyof` operator to get a union of the keys of the `myObject` object. We then use this union as a generic constraint on the `key` parameter to make sure that we can only pass valid keys of the object to the function.

We use the `typeof` operator to get the type of the `myObject` object, which is then used as the type of the `obj` parameter.


---

### Quick Check

**You want to create a new type that is a union of the keys of an existing type. Which of the following would you use?**

-> A. **`keyof`**
   B. `typeof`
   C. A mapped type
   D. A conditional type

<details>
<summary>See Answer</summary>

The `keyof` operator is the correct choice for this task. It takes an object type and returns a string or numeric literal union of its keys.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

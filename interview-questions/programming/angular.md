# Complete Angular Interview Guide

Master Angular with these real-world interview questions covering core concepts, dependency injection, and building scalable applications. Practice scenarios that mirror actual frontend engineering challenges.

**Companies that ask these questions:** Google | Microsoft | IBM | PayPal | Upwork

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What is the difference between a component and a directive i... | Conceptual | Core Concepts |
| 2 | What is dependency injection and how do you use it in Angula... | Conceptual | Dependency Injection |
| 3 | What is the component lifecycle and how do you use it in Ang... | Conceptual | Core Concepts |
| 4 | What is the difference between a template-driven and a react... | Conceptual | Forms |
| 5 | What is RxJS and how is it used in Angular? | Conceptual | RxJS |
| 6 | How do you do routing in Angular? | Practical | Routing |
| 7 | How do you do state management in Angular? | Architecture | State Management |
| 8 | What is change detection and how does it work in Angular? | Conceptual | Core Concepts |
| 9 | What are pipes and how do you use them in Angular? | Conceptual | Core Concepts |
| 10 | What is the difference between a module and a component in A... | Conceptual | Core Concepts |
| 11 | What are the different ways to communicate between component... | Conceptual | Components |
| 12 | How do you do server-side rendering (SSR) in Angular? | Practical | SSR |

---

## What You'll Learn

- Understand the core concepts of Angular.
- Master dependency injection.
- Build and deploy scalable applications.
- Debug and optimize Angular applications.
- Work with the Angular ecosystem of libraries and frameworks.

---

## Interview Questions

### Question 1: What is the difference between a component and a directive in Angular?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new feature that needs to add some custom behavior to an existing HTML element.

You are not sure whether to create a new component or a new directive.

## The Challenge

Explain the difference between a component and a directive in Angular. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might try to use a component to do something that could be more easily done with a directive.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between a component and a directive. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | Component                                                              | Directive                                                              |
| ------------ | ----------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Purpose**  | To create a new UI element with its own template and logic.             | To add some custom behavior to an existing HTML element.                 |
| **Template** | Has a template.                                                         | Does not have a template.                                              |
| **Syntax**   | `<my-component></my-component>`                                         | `<div my-directive></div>`                                             |
| **Use Cases**  | When you need to create a new UI element.                               | When you need to add some custom behavior to an existing HTML element.   |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use a **directive**. This is because we want to add some custom behavior to an existing HTML element, not create a new UI element.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**Component:**

```typescript


@Component({
  selector: 'my-component',
  template: '<h1>Hello, world!</h1>'
})
export class MyComponent {}
```

**Directive:**

```typescript


@Directive({
  selector: '[myDirective]'
})
export class MyDirective {
  constructor(el: ElementRef) {
    el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

---

### Question 2: What is dependency injection and how do you use it in Angular?

**Type:** Conceptual | **Category:** Dependency Injection

## The Scenario

You are a frontend engineer at a social media company. You are building a new component that needs to get data from a service.

You could create a new instance of the service in the component, but this would not be a very good solution. It would make the component difficult to test, and it would not be very flexible.

## The Challenge

Explain what dependency injection is in Angular and how you would use it to solve this problem. What are the key benefits of using dependency injection?


> **Common Mistake:** A junior engineer might not be aware of dependency injection. They might try to solve this problem by creating a new instance of the service in the component, which would be a very bad practice.

> **Senior Engineer Approach:** A senior engineer would know that dependency injection is the perfect tool for this job. They would be able to explain what dependency injection is and how to use it to provide a service to a component.

### Step 1: Understand What Dependency Injection Is

Dependency injection (DI) is a design pattern in which a class requests dependencies from external sources rather than creating them itself.

### Step 2: The Key Concepts of Dependency Injection

| Concept      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **Injector** | The injector is the main container for all the dependencies in an Angular application.                  |
| **Provider** | A provider is a recipe for creating a dependency.                                                       |
| **Dependency**| A dependency is an object that a class needs to perform its function.                                   |

### Step 3: Use Dependency Injection

Here's how we can use dependency injection to provide a service to a component:

**1. Create a service:**

```typescript


@Injectable({
  providedIn: 'root'
})
export class MyService {
  get_data() {
    return 'some data';
  }
}
```

**2. Inject the service into a component:**

```typescript


@Component({
  selector: 'my-component',
  template: '<h1>{{ myData }}</h1>'
})
export class MyComponent {
  myData: string;

  constructor(private myService: MyService) {
    this.myData = this.myService.get_data();
  }
}
```

In this example, we use the `constructor` to inject the `MyService` into the `MyComponent`.


---

### Quick Check

**You want to provide a different implementation of a service for a specific component. Which of the following would you use?**

   A. The `providedIn` property of the `@Injectable` decorator.
-> B. **The `providers` array of the `@Component` decorator.**
   C. The `providers` array of the `@NgModule` decorator.
   D. None of the above

<details>
<summary>See Answer</summary>

The `providers` array of the `@Component` decorator is the correct choice for this task. It allows you to provide a different implementation of a service for a specific component and its children.

</details>

---

### Question 3: What is the component lifecycle and how do you use it in Angular?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new component that needs to fetch data from an API when it is created.

You are not sure where to put the data fetching logic in the component's lifecycle.

## The Challenge

Explain what the component lifecycle is in Angular and what the different lifecycle hooks are. Which lifecycle hook would you use to fetch data from an API, and why?


> **Common Mistake:** A junior engineer might not be aware of the component lifecycle. They might try to fetch the data in the constructor, which would work, but it would not be the most optimal solution.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the component lifecycle and the different lifecycle hooks. They would also be able to explain the trade-offs between each hook and would have a clear recommendation for which one to use for fetching data.

### Step 1: Understand the Component Lifecycle

The component lifecycle is a series of stages that a component goes through, from when it is created to when it is destroyed.

### Step 2: The Different Lifecycle Hooks

| Hook                 | Description                                                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------- |
| **`ngOnChanges()`**  | Called before `ngOnInit()` and whenever one or more data-bound input properties change.                 |
| **`ngOnInit()`**     | Called once, after the first `ngOnChanges()`.                                                           |
| **`ngDoCheck()`**    | Called during every change detection run.                                                               |
| **`ngAfterContentInit()`**| Called after content (ng-content) has been projected into view.                                    |
| **`ngAfterContentChecked()`**| Called after the projected content has been checked.                                              |
| **`ngAfterViewInit()`**| Called after the component's view (and child views) has been initialized.                              |
| **`ngAfterViewChecked()`**| Called after the component's view (and child views) has been checked.                                |
| **`ngOnDestroy()`**  | Called just before the component is destroyed.                                                          |

### Step 3: Choose the Right Tool for the Job

For our use case, we should use the **`ngOnInit()` hook** to fetch the data from the API. This is because we want to make sure that the component has been initialized before we try to fetch the data.

### Step 4: Code Examples

Here's how we can use the `ngOnInit()` hook to fetch data from an API:

```typescript


@Component({
  selector: 'my-component',
  template: '<h1>{{ myData }}</h1>'
})
export class MyComponent implements OnInit {
  myData: string;

  constructor(private myService: MyService) {}

  ngOnInit() {
    this.myService.get_data().subscribe(data => {
      this.myData = data;
    });
  }
}
```


---

### Quick Check

**You want to perform an action just before a component is destroyed, such as cleaning up a timer. Which lifecycle hook would you use?**

   A. `ngOnInit()`
   B. `ngOnChanges()`
-> C. **`ngOnDestroy()`**
   D. `ngAfterViewInit()`

<details>
<summary>See Answer</summary>

`ngOnDestroy()` is the correct choice for this task. It is called just before the component is destroyed, so it is the perfect place to perform cleanup operations.

</details>

---

### Question 4: What is the difference between a template-driven and a reactive form in Angular?

**Type:** Conceptual | **Category:** Forms

## The Scenario

You are a frontend engineer at an e-commerce company. You are building a new form that allows users to create a new product.

You are not sure whether to use a template-driven form or a reactive form.

## The Challenge

Explain the difference between a template-driven and a reactive form in Angular. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in features or the design implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between a template-driven and a reactive form. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | Template-Driven Form                                                              | Reactive Form                                                              |
| ------------ | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Setup**    | Minimal setup required.                                                         | More setup required.                                                       |
| **Logic**    | The logic is in the template.                                                   | The logic is in the component class.                                       |
| **Testing**  | More difficult to test.                                                         | Easier to test.                                                            |
| **Flexibility**| Less flexible.                                                                  | More flexible.                                                             |
| **Use Cases**  | Simple forms with minimal validation.                                           | Complex forms with custom validation and dynamic fields.                   |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use a **reactive form**. This is because we are building a complex form with custom validation and dynamic fields.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**Template-Driven Form:**

```html
<form #myForm="ngForm">
  <input type="text" name="name" ngModel required>
  <button [disabled]="!myForm.valid">Submit</button>
</form>
```

**Reactive Form:**

```typescript


@Component({
  selector: 'my-form',
  template: `
    <form [formGroup]="myForm">
      <input type="text" formControlName="name">
      <button [disabled]="!myForm.valid">Submit</button>
    </form>
  `
})
export class MyFormComponent implements OnInit {
  myForm: FormGroup;

  ngOnInit() {
    this.myForm = new FormGroup({
      name: new FormControl('', Validators.required)
    });
  }
}
```


---

### Quick Check

**You are building a simple contact form with only a few fields and no validation. Which of the following would be the most appropriate?**

-> A. **A template-driven form**
   B. A reactive form
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

A template-driven form is the correct choice for this task. It is the simplest way to build a form in Angular, and it is perfectly suited for simple forms with minimal validation.

</details>

---

### Question 5: What is RxJS and how is it used in Angular?

**Type:** Conceptual | **Category:** RxJS

## The Scenario

You are a frontend engineer at a social media company. You are building a new feature that needs to handle a stream of real-time data from a WebSocket.

You need to find a way to process the data as it comes in, and you are considering using RxJS.

## The Challenge

Explain what RxJS is and how it is used in Angular. What are the key concepts of RxJS, and how would you use them to solve this problem?


> **Common Mistake:** A junior engineer might not be aware of RxJS. They might try to solve this problem by using a series of callbacks, which would be difficult to read and maintain.

> **Senior Engineer Approach:** A senior engineer would know that RxJS is the perfect tool for this job. They would be able to explain what RxJS is and how to use it to handle a stream of real-time data.

### Step 1: Understand What RxJS Is

RxJS is a library for reactive programming using Observables, to make it easier to compose asynchronous or callback-based code.

### Step 2: The Key Concepts of RxJS

| Concept      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **Observable**| An `Observable` is a stream of data that can be subscribed to.                                          |
| **Observer** | An `Observer` is an object with `next()`, `error()`, and `complete()` methods that is used to subscribe to an `Observable`. |
| **Operator** | An `Operator` is a function that can be used to transform, filter, or combine `Observable`s.               |
| **Subscription**| A `Subscription` represents the execution of an `Observable`. It can be used to unsubscribe from an `Observable`. |

### Step 3: How RxJS is Used in Angular

RxJS is used extensively in Angular for a variety of tasks, such as:

-   **HTTP requests:** The `HttpClient` in Angular returns an `Observable` that can be used to handle the response from an HTTP request.
-   **Forms:** The `FormControl` in Angular has a `valueChanges` property that is an `Observable` that can be used to listen for changes to the value of a form control.
-   **Routing:** The `Router` in Angular has an `events` property that is an `Observable` that can be used to listen for routing events.

### Step 4: Solve the Problem

Here's how we can use RxJS to handle the stream of real-time data from the WebSocket:

```typescript


const subject = webSocket('ws://localhost:8081');

subject.subscribe(
  (msg) => console.log('message received: ' + msg),
  (err) => console.log(err),
  () => console.log('complete')
);

subject.next(JSON.stringify({ op: 'hello' }));
```

In this example, we use the `webSocket` function from RxJS to create an `Observable` that represents the WebSocket connection. We then subscribe to the `Observable` to receive the data as it comes in.


---

### Quick Check

**You want to make two HTTP requests in parallel and then do something with the results of both requests. Which of the following would you use?**

   A. `concat`
   B. `merge`
-> C. **`forkJoin`**
   D. `zip`

<details>
<summary>See Answer</summary>

`forkJoin` is the correct choice for this task. It will wait for both HTTP requests to complete and then it will emit an array with the results of both requests.

</details>

---

### Question 6: How do you do routing in Angular?

**Type:** Practical | **Category:** Routing

## The Scenario

You are a frontend engineer at a social media company. You are building a new single-page application (SPA) that will have multiple pages, such as a home page, a profile page, and a settings page.

You need to find a way to handle the routing between these pages.

## The Challenge

Explain how you would do routing in Angular. What is the Angular Router, and what are the key features of it?


> **Common Mistake:** A junior engineer might try to solve this problem by using a series of `*ngIf` statements to show and hide the different pages. This would not be a very scalable or maintainable solution.

> **Senior Engineer Approach:** A senior engineer would know that the Angular Router is the perfect tool for this job. They would be able to explain what the Angular Router is and how to use it to handle the routing between the different pages.

### Step 1: Understand What the Angular Router Is

The Angular Router is a powerful router that is built and maintained by the Angular team. It allows you to map URLs to components, so that you can build a single-page application with multiple pages.

### Step 2: The Key Features of the Angular Router

| Feature          | Description                                                                                             |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **Nested Routes**| You can nest routes inside other routes to create complex layouts.                                      |
| **Dynamic Routes**| You can use dynamic segments in your routes to match a pattern, such as `/user/:id`.                    |
| **Route Guards** | You can use route guards to protect routes from unauthorized access.                                    |
| **Lazy Loading**  | You can lazy load modules to improve the performance of your application.                               |

### Step 3: Set Up the Angular Router

Here's how we can set up the Angular Router in our application:

**1. Import the `RouterModule`:**

```typescript


const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'profile', component: ProfileComponent },
  { path: 'settings', component: SettingsComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

**2. Add the `<router-outlet>` component to your template:**

The `<router-outlet>` component is where the component for the current route will be rendered.

```html
<router-outlet></router-outlet>
```

**3. Use the `routerLink` directive to navigate between routes:**

```html
<a routerLink="/">Home</a>
<a routerLink="/profile">Profile</a>
<a routerLink="/settings">Settings</a>
```


---

### Quick Check

**You want to protect a route so that it can only be accessed by authenticated users. Which of the following would you use?**

   A. A nested route
   B. A dynamic route
-> C. **A route guard**
   D. Lazy loading

<details>
<summary>See Answer</summary>

A route guard is the correct choice for this task. You can use a route guard to check if the user is authenticated before allowing them to access the route.

</details>

---

### Question 7: How do you do state management in Angular?

**Type:** Architecture | **Category:** State Management

## The Scenario

You are a frontend engineer at a social media company. You are building a new single-page application (SPA) that has a complex state that needs to be shared across multiple components.

You are not sure whether to use a simple service or a more advanced state management library like NgRx.

## The Challenge

Explain how you would do state management in Angular. What are the different state management patterns that you would use, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might try to solve this problem by using a service with a `BehaviorSubject`. This would work for a small application, but it would not be a very scalable or maintainable solution for a large application.

> **Senior Engineer Approach:** A senior engineer would know that a state management library like NgRx is the perfect tool for this job. They would be able to explain the different state management patterns and would have a clear plan for how to use NgRx to manage the state of a large application.

### Step 1: Understand the Different State Management Patterns

| Pattern               | Description                                                                                             | Pros                                                              | Cons                                                               |
| --------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Simple Service**    | A simple service with a `BehaviorSubject` that can be used to store and share the state.            | Easy to understand and implement.                                 | Can become difficult to maintain for a large application.          |
| **NgRx**              | A state management library for Angular applications. It is inspired by Redux and provides a more structured and maintainable way to manage the state of a large application. | Provides a more structured and maintainable way to manage the state of a large application. | Can be more complex to set up than a simple service.               |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use **NgRx**. This is because we are building a large application with a complex state that needs to be shared across multiple components.

### Step 3: Code Examples

Here's how we can use NgRx to manage the state of our application:

**1. Define the state, actions, and reducer:**

```typescript
// app.state.ts
export interface AppState {
  count: number;
}

// app.actions.ts


export const increment = createAction('[Counter] Increment');

// app.reducer.ts


export const initialState = 0;

const _counterReducer = createReducer(
  initialState,
  on(increment, (state) => state + 1)
);

export function counterReducer(state, action) {
  return _counterReducer(state, action);
}
```

**2. Use the store in a component:**

```typescript


@Component({
  selector: 'my-counter',
  template: `
    <p>{{ count$ | async }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class MyCounterComponent {
  count$: Observable<number>;

  constructor(private store: Store<{ count: number }>) {
    this.count$ = store.pipe(select('count'));
  }

  increment() {
    this.store.dispatch(increment());
  }
}
```


---

### Quick Check

**You want to be able to access a piece of derived state from your store. Which of the following would you use?**

   A. An action
   B. A reducer
-> C. **A selector**
   D. An effect

<details>
<summary>See Answer</summary>

A selector is the correct choice for this task. It is a pure function that takes the state as input and returns a piece of derived state.

</details>

---

### Question 8: What is change detection and how does it work in Angular?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new component that displays a real-time feed of data.

The component is not updating as quickly as you would like, and you suspect that the issue might be with the change detection strategy.

## The Challenge

Explain what change detection is in Angular and how it works. What are the different change detection strategies, and how would you use them to optimize the performance of the component?


> **Common Mistake:** A junior engineer might not be aware of change detection. They might try to solve the problem by manually updating the DOM, which would be a very bad practice.

> **Senior Engineer Approach:** A senior engineer would know that change detection is a critical part of the Angular framework. They would be able to explain what change detection is and how it works. They would also have a clear plan for how to use the different change detection strategies to optimize the performance of the component.

### Step 1: Understand What Change Detection Is

Change detection is the process of detecting when the state of a component has changed and then re-rendering the component to reflect the new state.

### Step 2: The Different Change Detection Strategies

| Strategy          | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **`Default`**     | The default change detection strategy. Angular will check for changes on every browser event, timer, and XHR. |
| **`OnPush`**      | A more performant change detection strategy. Angular will only check for changes when an input property of the component changes, or when an event is fired by the component or one of its children. |

### Step 3: Choose the Right Tool for the Job

For our use case, we should use the **`OnPush` change detection strategy**. This is because we are building a component that displays a real-time feed of data, and we want to make sure that the component is only re-rendered when the data actually changes.

### Step 4: Code Examples

Here's how we can use the `OnPush` change detection strategy:

```typescript


@Component({
  selector: 'my-component',
  template: '...',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class MyComponent {
  // ...
}
```

By using the `OnPush` change detection strategy, we can significantly improve the performance of our component.


---

### Quick Check

**You have a component that has a lot of input properties that change frequently. Which change detection strategy would be the most appropriate?**

-> A. **`Default`**
   B. `OnPush`
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

The `Default` change detection strategy is the correct choice for this task. The `OnPush` strategy would not be a good choice, because the component would not be re-rendered when the input properties change.

</details>

---

### Question 9: What are pipes and how do you use them in Angular?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new component that needs to display a date in a user-friendly format.

You could format the date in the component's class, but this would not be a very reusable solution.

## The Challenge

Explain what pipes are in Angular and how you would use them to solve this problem. What are the key benefits of using pipes?


### Step 1: Understand What Pipes Are

A pipe is a way to transform data in your template. It takes in data as input and returns a transformed version of that data.

### Step 2: The Different Types of Pipes

| Type        | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **Pure**    | A pure pipe is only called when its input changes. This is the default.                                 |
| **Impure**  | An impure pipe is called on every change detection cycle.                                                 |

### Step 3: Use a Built-in Pipe

Angular has a number of built-in pipes that you can use, such as:

-   `DatePipe`
-   `UpperCasePipe`
-   `LowerCasePipe`
-   `CurrencyPipe`
-   `DecimalPipe`
-   `PercentPipe`

Here's how we can use the `DatePipe` to format a date:

```html
{{ myDate | date:'medium' }}
```

### Step 4: Create a Custom Pipe

You can also create your own custom pipes. Here's how we can create a custom pipe to truncate a string:

```typescript


@Pipe({
  name: 'truncate'
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number): string {
    return value.length < limit ? value : value.slice(0, limit) + '...';
  }
}
```

We can then use the custom pipe in our template:

```html
{{ myString | truncate:10 }}
```


---

### Quick Check

**You are building a component that displays a list of items that are being filtered in real-time. Which type of pipe would you use for the filter?**

   A. A pure pipe
-> B. **An impure pipe**
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

An impure pipe is the correct choice for this task. A pure pipe would not be re-evaluated when the filter criteria changes.

</details>

---

### Question 10: What is the difference between a module and a component in Angular?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new feature that will be used in multiple places in the application.

You are not sure whether to create a new module or a new component.

## The Challenge

Explain the difference between a module and a component in Angular. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in purpose or the design implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between a module and a component. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | Component                                                              | Module                                                              |
| ------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Purpose**  | To create a new UI element with its own template and logic.             | To group related components, directives, pipes, and services together. |
| **Template** | Has a template.                                                         | Does not have a template.                                           |
| **Syntax**   | `@Component`                                                            | `@NgModule`                                                         |
| **Use Cases**  | When you need to create a new UI element.                               | When you need to organize your application into logical blocks of functionality. |

### Step 2: Choose the Right Tool for the Job

For our use case, we should create a new **module** that contains the new component. This will allow us to easily reuse the component in multiple places in the application.

### Step 3: Code Examples

Here's how we can create a new module and a new component:

**`my-feature.module.ts`:**

```typescript


@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [
    MyFeatureComponent
  ],
  exports: [
    MyFeatureComponent
  ]
})
export class MyFeatureModule { }
```

**`my-feature.component.ts`:**

```typescript


@Component({
  selector: 'my-feature',
  template: '<h1>My Feature</h1>'
})
export class MyFeatureComponent {}
```


---

### Quick Check

**You are building a new application and you want to organize your code into logical blocks of functionality. Which of the following would you use?**

   A. A component
   B. A directive
-> C. **A module**
   D. A service

<details>
<summary>See Answer</summary>

A module is the correct choice for this task. It allows you to group related components, directives, pipes, and services together, which makes it easy to organize your application into logical blocks of functionality.

</details>

---

### Question 11: What are the different ways to communicate between components in Angular?

**Type:** Conceptual | **Category:** Components

## The Scenario

You are a frontend engineer at a social media company. You are building a new feature that consists of two components: a parent component and a child component.

You need to find a way to pass data from the parent component to the child component, and you also need to find a way to notify the parent component when an event occurs in the child component.

## The Challenge

Explain the different ways to communicate between components in Angular. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might try to solve this problem by using a shared service. This would work, but it would be overkill for this simple use case.

> **Senior Engineer Approach:** A senior engineer would know that there are several different ways to communicate between components in Angular. They would be able to explain the pros and cons of each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Communication Patterns

| Pattern      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **`@Input()`** | To pass data from a parent component to a child component.                                              |
| **`@Output()`**| To emit an event from a child component to a parent component.                                          |
| **ViewChild**| To get a reference to a child component in the parent component.                                        |
| **Service**  | To share data between components that are not directly related.                                         |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use a combination of **`@Input()` and `@Output()`**.

-   We will use `@Input()` to pass data from the parent component to the child component.
-   We will use `@Output()` to emit an event from the child component to the parent component.

### Step 3: Code Examples

Here's how we can use `@Input()` and `@Output()` to communicate between a parent and a child component:

**`child.component.ts`:**

```typescript


@Component({
  selector: 'my-child',
  template: `
    <p>{{ myInput }}</p>
    <button (click)="myEvent.emit('hello from child')">Click me</button>
  `
})
export class MyChildComponent {
  @Input() myInput: string;
  @Output() myEvent = new EventEmitter<string>();
}
```

**`parent.component.ts`:**

```typescript


@Component({
  selector: 'my-parent',
  template: `
    <my-child [myInput]="'hello from parent'" (myEvent)="onMyEvent($event)"></my-child>
  `
})
export class MyParentComponent {
  onMyEvent(message: string) {
    console.log(message);
  }
}
```


---

### Quick Check

**You want to share data between two components that are not directly related. Which of the following would you use?**

   A. `@Input()`
   B. `@Output()`
   C. ViewChild
-> D. **A service**

<details>
<summary>See Answer</summary>

A service is the correct choice for this task. It allows you to share data between components that are not directly related by injecting the service into both components.

</details>

---

### Question 12: How do you do server-side rendering (SSR) in Angular?

**Type:** Practical | **Category:** SSR

## The Scenario

You are a frontend engineer at an e-commerce company. You are building a new public-facing website that needs to be fast and SEO-friendly.

You are considering using server-side rendering (SSR) to improve the performance and SEO of the website.

## The Challenge

Explain what server-side rendering (SSR) is in Angular and how you would use it to solve this problem. What are the key benefits of using SSR?


> **Common Mistake:** A junior engineer might not be aware of SSR. They might try to solve this problem by pre-rendering the pages, which would not be a very scalable solution.

> **Senior Engineer Approach:** A senior engineer would know that SSR is the perfect tool for this job. They would be able to explain what SSR is and how to use it to improve the performance and SEO of a website. They would also be aware of Angular Universal, which is the official way to do SSR with Angular.

### Step 1: Understand What SSR Is

Server-side rendering (SSR) is the process of rendering a single-page application (SPA) on the server and then sending the fully rendered page to the client.

### Step 2: The Benefits of Using SSR

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Performance**   | SSR can improve the performance of a website by reducing the time to first contentful paint.           |
| **SEO**           | SSR can improve the SEO of a website by making it easier for search engine crawlers to index the content. |

### Step 3: How to Do SSR with Angular Universal

Angular Universal is the official way to do SSR with Angular. Here's how you can add it to your existing Angular application:

```bash
ng add @nguniversal/express-engine
```

This command will automatically add the necessary files and configurations to your project.

### Step 4: Build and Run the Application

Once you have added Angular Universal to your project, you can build and run the application with the following commands:

```bash
npm run build:ssr
npm run serve:ssr
```

This will start a Node.js Express server that will render your application on the server.


---

### Quick Check

**You are building a new blog and you want it to be as fast and SEO-friendly as possible. Which of the following would be the most appropriate?**

   A. A client-side rendered (CSR) application
   B. A server-side rendered (SSR) application
-> C. **A statically generated site (SSG)**
   D. Any of the above

<details>
<summary>See Answer</summary>

A statically generated site (SSG) is the correct choice for this task. It will pre-render all the pages at build time, which will make the website very fast and SEO-friendly. You can use Angular Universal to create a statically generated site.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

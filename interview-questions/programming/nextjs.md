# Complete Next.js Interview Guide

Master Next.js with these real-world interview questions covering core concepts, server-side rendering, and building scalable applications. Practice scenarios that mirror actual frontend engineering challenges.

**Companies that ask these questions:** Vercel | Netflix | Twitch | Hulu | Nike

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What is the difference between server-side rendering (SSR) a... | Conceptual | Core Concepts |
| 2 | How do you do data fetching in Next.js? | Practical | Data Fetching |
| 3 | How do you do routing in Next.js? | Practical | Routing |
| 4 | What are API routes and how do you use them in Next.js? | Practical | API Routes |
| 5 | How do you do styling in Next.js? | Conceptual | Styling |
| 6 | Your Next.js application is slow. How do you optimize its pe... | Debugging | Performance Optimization |
| 7 | What is the `_app.js` file and what is it used for in Next.j... | Conceptual | Core Concepts |
| 8 | What is the `_document.js` file and what is it used for in N... | Conceptual | Core Concepts |
| 9 | How do you handle authentication and authorization in Next.j... | Practical | Security |
| 10 | How do you handle environment variables in Next.js? | Practical | Configuration |
| 11 | How do you deploy a Next.js application? | Practical | Deployment |
| 12 | What is middleware and how do you use it in Next.js? | Practical | Middleware |

---

## What You'll Learn

- Understand the core concepts of Next.js.
- Master server-side rendering and static site generation.
- Build and deploy scalable applications.
- Debug and optimize Next.js applications.
- Work with the Next.js ecosystem of libraries and frameworks.

---

## Interview Questions

### Question 1: What is the difference between server-side rendering (SSR) and static site generation (SSG) in Next.js?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at an e-commerce company. You are building a new public-facing website that will have a product catalog.

You are not sure whether to use server-side rendering (SSR) or static site generation (SSG) to build the website.

## The Challenge

Explain the difference between server-side rendering (SSR) and static site generation (SSG) in Next.js. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the difference in performance or the design implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between SSR and SSG. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | Server-Side Rendering (SSR)                                                              | Static Site Generation (SSG)                                                              |
| ------------ | --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Rendering**| The page is rendered on the server for each request.                                    | The page is rendered at build time.                                                      |
| **Performance**| Slower than SSG, because the page has to be rendered for each request.                  | Faster than SSR, because the page is already rendered.                                   |
| **Data**     | The data is always up-to-date.                                                          | The data can be stale.                                                                   |
| **Use Cases**  | When you have a page that needs to be personalized for each user, or when you have a page that has a lot of dynamic data. | When you have a page that does not change often, such as a blog post or a product catalog. |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use **SSG**. This is because the product catalog does not change very often, and we want the website to be as fast as possible.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**SSR:**

```javascript
export async function getServerSideProps(context) {
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  return { props: { data } }
}
```

**SSG:**

```javascript
export async function getStaticProps(context) {
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  return { props: { data } }
}
```


---

### Quick Check

**You are building a new blog and you want it to be as fast and SEO-friendly as possible. Which of the following would be the most appropriate?**

   A. A client-side rendered (CSR) application
   B. A server-side rendered (SSR) application
-> C. **A statically generated site (SSG)**
   D. Any of the above

<details>
<summary>See Answer</summary>

A statically generated site (SSG) is the correct choice for this task. It will pre-render all the pages at build time, which will make the website very fast and SEO-friendly.

</details>

---

### Question 2: How do you do data fetching in Next.js?

**Type:** Practical | **Category:** Data Fetching

## The Scenario

You are a frontend engineer at a social media company. You are building a new page that needs to fetch data from an API and then render it.

You are not sure which of the different data fetching methods in Next.js you should use.

## The Challenge

Explain the different data fetching methods in Next.js. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might only be aware of `getInitialProps`. They might not be aware of the other data fetching methods or the trade-offs between them.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of all the different data fetching methods in Next.js. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Different Data Fetching Methods

| Method                 | Description                                                                                             |
| ---------------------- | ------------------------------------------------------------------------------------------------------- |
| **`getStaticProps`**   | Fetches data at build time.                                                                             |
| **`getStaticPaths`**   | Specifies which dynamic routes to pre-render at build time.                                             |
| **`getServerSideProps`**| Fetches data on each request.                                                                           |
| **`getInitialProps`**  | An older data fetching method that runs on both the server and the client. It is not recommended for new projects. |
| **Client-side fetching** | Fetches data on the client.                                                                             |

### Step 2: Choose the Right Tool for the Job

| Use Case                               | Recommended Method |
| -------------------------------------- | ------------------ |
| A blog post or a product catalog       | `getStaticProps`   |
| A page with a large number of dynamic routes | `getStaticPaths`   |
| A page with personalized data          | `getServerSideProps`|
| A page with real-time data             | Client-side fetching |

### Step 3: Code Examples

Here are some code examples that show the difference between the different approaches:

**`getStaticProps`:**

```javascript
export async function getStaticProps() {
  const res = await fetch('https://.../data')
  const data = await res.json()

  return { props: { data } }
}
```

**`getServerSideProps`:**

```javascript
export async function getServerSideProps() {
  const res = await fetch('https://.../data')
  const data = await res.json()

  return { props: { data } }
}
```

**Client-side fetching:**

```javascript


function MyComponent() {
  const [data, setData] = useState(null)

  useEffect(() => {
    async function fetchData() {
      const res = await fetch('https://.../data')
      const data = await res.json()
      setData(data)
    }
    fetchData()
  }, [])

  // ...
}
```


---

### Quick Check

**You are building a page that displays a list of products. The list of products changes frequently. Which of the following would be the most appropriate?**

   A. `getStaticProps`
-> B. **`getServerSideProps`**
   C. Client-side fetching
   D. Any of the above

<details>
<summary>See Answer</summary>

`getServerSideProps` is the correct choice for this task. It will fetch the data on each request, so the list of products will always be up-to-date.

</details>

---

### Question 3: How do you do routing in Next.js?

**Type:** Practical | **Category:** Routing

## The Scenario

You are a frontend engineer at a social media company. You are building a new application that will have multiple pages, such as a home page, a profile page, and a settings page.

You need to find a way to handle the routing between these pages.

## The Challenge

Explain how you would do routing in Next.js. What is the file-system based router, and what are the key features of it?


> **Common Mistake:** A junior engineer might try to solve this problem by using a library like React Router. This would work, but it would not be the standard way to do routing in Next.js.

> **Senior Engineer Approach:** A senior engineer would know that Next.js has a built-in file-system based router. They would be able to explain how it works and how to use it to handle the routing between the different pages.

### Step 1: Understand What the File-System Based Router Is

Next.js has a file-system based router. This means that you can create a new route by just creating a new file in the `pages` directory.

### Step 2: The Key Features of the File-System Based Router

| Feature          | Description                                                                                             |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| **Index Routes** | The file `pages/index.js` is mapped to the `/` route.                                                  |
| **Nested Routes**| The file `pages/about/us.js` is mapped to the `/about/us` route.                                       |
| **Dynamic Routes**| The file `pages/posts/[id].js` is mapped to the `/posts/:id` route.                                      |

### Step 3: Code Examples

Here are some code examples that show how to create different types of routes:

**Index Route:**

```javascript
// pages/index.js
export default function Home() {
  return <h1>Home Page</h1>;
}
```

**Nested Route:**

```javascript
// pages/about/us.js
export default function AboutUs() {
  return <h1>About Us Page</h1>;
}
```

**Dynamic Route:**

```javascript
// pages/posts/[id].js


export default function Post() {
  const router = useRouter();
  const { id } = router.query;

  return <h1>Post: {id}</h1>;
}
```

### The `Link` Component

You can use the `Link` component from `next/link` to navigate between routes.

```javascript


export default function Home() {
  return (
    <div>
      <Link href="/about/us">
        <a>About Us</a>
      </Link>
    </div>
  );
}
```


<InterviewQuiz
  question="You want to create a new route for `/products/123`. Which of the following would you use?"
  options={[
    "`pages/products/123.js`",
    "`pages/products/[id].js`",
    "Either is fine",
    "Neither is suitable for this task"
  ]}
  correctAnswer={1}
  explanation="`pages/products/[id].js` is the correct choice for this task. It is a dynamic route that will match any URL that starts with `/products/` and has a second segment."
/>

---

### Question 4: What are API routes and how do you use them in Next.js?

**Type:** Practical | **Category:** API Routes

## The Scenario

You are a frontend engineer at a social media company. You are building a new feature that needs to have a simple backend API to store and retrieve data.

You could create a separate backend application, but this would be overkill for this simple use case.

## The Challenge

Explain what API routes are in Next.js and how you would use them to solve this problem. What are the key benefits of using API routes?


> **Common Mistake:** A junior engineer might not be aware of API routes. They might try to solve this problem by creating a separate backend application, which would be a more complex and time-consuming solution.

> **Senior Engineer Approach:** A senior engineer would know that API routes are the perfect tool for this job. They would be able to explain what API routes are and how to use them to build a simple backend API.

### Step 1: Understand What API Routes Are

API routes are a feature of Next.js that allows you to create a backend API inside your Next.js application.

### Step 2: Create an API Route

To create an API route, you just need to create a new file in the `pages/api` directory.

Here's how we can create a simple API route that returns a list of users:

```javascript
// pages/api/users.js
export default function handler(req, res) {
  res.status(200).json([
    { id: 1, name: 'John Doe' },
    { id: 2, name: 'Jane Doe' }
  ]);
}
```

This API route will be available at `/api/users`.

### Dynamic API Routes

You can also create dynamic API routes. For example, the file `pages/api/users/[id].js` will be mapped to the `/api/users/:id` route.

```javascript
// pages/api/users/[id].js
export default function handler(req, res) {
  const { id } = req.query;
  res.status(200).json({ id, name: `User ${id}` });
}
```

### The Benefits of Using API Routes

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Simplicity**    | API routes are very easy to use.                                                                        |
| **Integration**   | API routes are tightly integrated with the Next.js ecosystem.                                           |
| **Scalability**   | API routes are serverless functions, which means that they can be scaled automatically.                 |


---

### Quick Check

**You want to create a new API route that can handle `POST` requests. Which of the following would you use?**

-> A. **The `req.method` property**
   B. The `res.status()` method
   C. The `res.json()` method
   D. None of the above

<details>
<summary>See Answer</summary>

The `req.method` property is the correct choice for this task. You can use it to check the HTTP method of the request and then handle the request accordingly.

</details>

---

### Question 5: How do you do styling in Next.js?

**Type:** Conceptual | **Category:** Styling

## The Scenario

You are a frontend engineer at a social media company. You are building a new component that needs to have some custom styles.

You are not sure which of the different ways to do styling in Next.js you should use.

## The Challenge

Explain the different ways to do styling in Next.js. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might just use a global stylesheet for everything. This would not be a very scalable or maintainable solution.

> **Senior Engineer Approach:** A senior engineer would know that there are several different ways to do styling in Next.js. They would be able to explain the pros and cons of each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Different Ways to Do Styling

| Method             | Description                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| **Global Styles**  | A global stylesheet that is applied to the entire application.                                          |
| **CSS Modules**    | A CSS file in which all class names and animation names are scoped locally by default.                  |
| **Styled-JSX**     | A CSS-in-JS library that is built into Next.js.                                                         |
| **CSS-in-JS Libraries**| Libraries like Emotion and Styled Components.                                                       |

### Step 2: Choose the Right Tool for the Job

| Use Case                               | Recommended Method |
| -------------------------------------- | ------------------ |
| A simple application with a few pages  | Global Styles      |
| A component library                    | CSS Modules        |
| A large application with many components| CSS-in-JS Libraries|

For our use case, we should use **CSS Modules**. This is because we are building a new component that needs to have its own custom styles, and we want to make sure that the styles are scoped locally to the component.

### Step 3: Code Examples

Here are some code examples that show the difference between the different approaches:

**Global Styles:**

```javascript
// pages/_app.js


export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

**CSS Modules:**

```javascript
// components/MyComponent.module.css
.my-class {
  color: red;
}

// components/MyComponent.js


export default function MyComponent() {
  return <div className={styles['my-class']}>Hello, world!</div>
}
```

**Styled-JSX:**

```javascript
export default function MyComponent() {
  return (
    <div>
      <p>Hello, world!</p>
      <style jsx>{`
        p {
          color: red;
        }
      `}</style>
    </div>
  );
}
```


---

### Quick Check

**You are building a component library and you want to make sure that the styles are scoped locally to each component. Which of the following would be the most appropriate?**

   A. Global Styles
-> B. **CSS Modules**
   C. Styled-JSX
   D. Any of the above

<details>
<summary>See Answer</summary>

CSS Modules is the correct choice for this task. It automatically scopes all class names and animation names locally to each component, which prevents a lot of common styling issues.

</details>

---

### Question 6: Your Next.js application is slow. How do you optimize its performance?

**Type:** Debugging | **Category:** Performance Optimization

## The Scenario

You are a frontend engineer at an e-commerce company. You are responsible for a Next.js application that is experiencing performance issues. The application is slow to load, and it is not very responsive.

You need to find a way to optimize the performance of the application.

## The Challenge

Explain your strategy for optimizing the performance of a Next.js application. What are the key areas that you would focus on, and what tools would you use to help you?


> **Common Mistake:** A junior engineer might not have a clear strategy for optimizing the performance of a Next.js application. They might try to solve the problem by just making random changes to the code, which would not be very effective.

> **Senior Engineer Approach:** A senior engineer would have a clear strategy for optimizing the performance of a Next.js application. They would be able to explain the key areas to focus on, and they would have a clear plan for how to use tools like the Next.js Analytics and the webpack-bundle-analyzer to identify and fix performance bottlenecks.

### Step 1: Analyze the Performance of the Application

The first step is to analyze the performance of the application. We can use the Next.js Analytics to do this. The Next.js Analytics provides a number of metrics that can be used to measure the performance of the application, such as the Time to First Byte (TTFB) and the First Contentful Paint (FCP).

### Step 2: The Key Areas to Focus On

| Area                | Description                                                                                             |
| ------------------- | ------------------------------------------------------------------------------------------------------- |
| **Bundle Size**     | A large bundle size can slow down the initial load time of the application.                             |
| **Render Performance**| A slow render performance can make the application feel unresponsive.                                  |
| **Data Fetching**   | A slow data fetching performance can delay the rendering of the page.                                   |

### Step 3: Optimize the Bundle Size

Here are some techniques we can use to optimize the bundle size:

-   **Code splitting:** Split the code into smaller chunks that can be loaded on demand.
-   **Tree shaking:** Remove unused code from the bundle.
-   **Lazy loading:** Lazy load components and libraries to improve the initial load time.

We can use the `@next/bundle-analyzer` to visualize the size of the bundle and to identify any large dependencies.

### Step 4: Optimize the Render Performance

Here are some techniques we can use to optimize the render performance:

-   **Use `React.memo`** to memoize functional components.
-   **Use `useMemo` and `useCallback`** to memoize values and functions.
-   **Use `next/image`** to optimize images.

### Step 5: Optimize the Data Fetching Performance

Here are some techniques we can use to optimize the data fetching performance:

-   **Use `getStaticProps`** for pages that do not change often.
-   **Use `getServerSideProps`** for pages that have personalized data.
-   **Use Incremental Static Regeneration (ISR)** to re-generate static pages in the background.


---

### Quick Check

**You are building a page that displays a list of products. The list of products is the same for all users and does not change often. Which of the following would be the most appropriate?**

-> A. **`getStaticProps`**
   B. `getServerSideProps`
   C. Client-side fetching
   D. Any of the above

<details>
<summary>See Answer</summary>

`getStaticProps` is the correct choice for this task. It will render the page at build time, which will make it very fast and SEO-friendly.

</details>

---

### Question 7: What is the `_app.js` file and what is it used for in Next.js?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new Next.js application and you want to add a global stylesheet that will be applied to the entire application.

You are not sure where to put the global stylesheet.

## The Challenge

Explain what the `_app.js` file is in Next.js and how you would use it to solve this problem. What are the key benefits of using the `_app.js` file?


> **Common Mistake:** A junior engineer might not be aware of the `_app.js` file. They might try to solve this problem by adding a `<link>` tag to each page, which would not be a very good solution.

> **Senior Engineer Approach:** A senior engineer would know that the `_app.js` file is the perfect tool for this job. They would be able to explain what the `_app.js` file is and how to use it to add a global stylesheet to the application.

### Step 1: Understand What the `_app.js` File Is

The `pages/_app.js` file is a special file in Next.js that allows you to override the default `App` component.

### Step 2: The Key Use Cases for the `_app.js` File

| Use Case                               | Description                                                                                             |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Global Styles**                      | To add a global stylesheet that will be applied to the entire application.                              |
| **Layouts**                            | To create a layout that will be shared across all the pages in the application.                         |
| **State Management**                   | To wrap the application in a state management library, such as Redux or MobX.                           |
| **Error Handling**                     | To add a global error boundary to the application.                                                      |

### Step 3: Solve the Problem

Here's how we can use the `_app.js` file to add a global stylesheet to our application:

```javascript
// pages/_app.js


export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

In this example, we import the global stylesheet in the `_app.js` file. This will cause the stylesheet to be applied to every page in the application.


---

### Quick Check

**You want to create a layout that will be shared across all the pages in your application. Which of the following would you use?**

   A. A component
   B. A directive
-> C. **The `_app.js` file**
   D. The `_document.js` file

<details>
<summary>See Answer</summary>

The `_app.js` file is the correct choice for this task. It allows you to wrap the `Component` prop in a layout component, which will be shared across all the pages in the application.

</details>

---

### Question 8: What is the `_document.js` file and what is it used for in Next.js?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a frontend engineer at a social media company. You are building a new Next.js application and you want to add some custom `<html>` and `<body>` tags to your application.

You are not sure where to put these tags.

## The Challenge

Explain what the `_document.js` file is in Next.js and how you would use it to solve this problem. What are the key benefits of using the `_document.js` file?


> **Common Mistake:** A junior engineer might not be aware of the `_document.js` file. They might try to solve this problem by adding the tags to each page, which would not be a very good solution.

> **Senior Engineer Approach:** A senior engineer would know that the `_document.js` file is the perfect tool for this job. They would be able to explain what the `_document.js` file is and how to use it to add custom `<html>` and `<body>` tags to the application.

### Step 1: Understand What the `_document.js` File Is

The `pages/_document.js` file is a special file in Next.js that allows you to override the default `Document` component.

### Step 2: The Key Use Cases for the `_document.js` File

| Use Case                               | Description                                                                                             |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Custom `<html>` and `<body>` tags** | To add custom `<html>` and `<body>` tags to your application.                                          |
| **Server-side rendering**              | To add server-side rendering logic to your application.                                                 |

### Step 3: Solve the Problem

Here's how we can use the `_document.js` file to add custom `<html>` and `<body>` tags to our application:

```javascript
// pages/_document.js


class MyDocument extends Document {
  render() {
    return (
      <Html lang="en">
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

export default MyDocument
```

In this example, we have added a `lang="en"` attribute to the `<html>` tag.

### `_app.js` vs. `_document.js`

| File            | Purpose                                                                                             |
| --------------- | --------------------------------------------------------------------------------------------------- |
| **`_app.js`**   | To initialize pages. This is the place to add a global stylesheet or a layout component.          |
| **`_document.js`**| To augment the `<html>` and `<body>` tags. This is the place to add a `lang` attribute or a custom font. |


---

### Quick Check

**You want to add a global stylesheet to your application. Which of the following would you use?**

-> A. **The `_app.js` file**
   B. The `_document.js` file
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

The `_app.js` file is the correct choice for this task. It is the place to add a global stylesheet or a layout component.

</details>

---

### Question 9: How do you handle authentication and authorization in Next.js?

**Type:** Practical | **Category:** Security

## The Scenario

You are a frontend engineer at a social media company. You are building a new application that needs to have a secure authentication and authorization system.

You need to find a way to protect your pages and API routes from unauthorized access.

## The Challenge

Explain how you would handle authentication and authorization in Next.js. What are the different strategies that you would use, and what are the trade-offs between them?


### Step 1: Understand the Different Strategies

| Strategy      | Description                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| **Cookie-based**| Store the user's session in a cookie.                                                                   |
| **Token-based**| Store the user's session in a JSON Web Token (JWT).                                                     |
| **Third-party**| Use a third-party authentication provider, such as Auth0 or Okta.                                       |

### Step 2: Choose the Right Tool for the Job

| Use Case                               | Recommended Strategy |
| -------------------------------------- | ------------------ |
| A simple application with a few users  | Cookie-based       |
| A large application with many users    | Token-based        |
| An enterprise application              | Third-party        |

For our use case, we should use a **token-based** strategy. This is because we are building a large application with many users, and we need a scalable and secure solution.

### Step 3: Implement Token-based Authentication

Here's how we can implement token-based authentication with Next.js:

**1. Create an API route for logging in:**

This API route will take a username and password, and it will return a JWT if the credentials are valid.

**2. Store the JWT on the client:**

The JWT can be stored in a cookie or in local storage.

**3. Send the JWT with each request:**

The JWT should be sent with each request in the `Authorization` header.

**4. Protect pages and API routes:**

We can use a higher-order component (HOC) or a middleware to protect our pages and API routes from unauthorized access.


---

### Quick Check

**You are building a simple application with a few users. Which of the following would be the most appropriate?**

-> A. **Cookie-based**
   B. Token-based
   C. Third-party
   D. Any of the above

<details>
<summary>See Answer</summary>

Cookie-based authentication is the correct choice for this task. It is the simplest way to handle authentication, and it is perfectly suited for a simple application with a few users.

</details>

---

### Question 10: How do you handle environment variables in Next.js?

**Type:** Practical | **Category:** Configuration

## The Scenario

You are a frontend engineer at a social media company. You are building a new Next.js application that needs to connect to a database.

You need to store the database credentials in a secure way, and you do not want to commit them to your Git repository.

## The Challenge

Explain how you would handle environment variables in Next.js. What are the different ways to set environment variables, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might try to hardcode the database credentials in the code. This would be a very bad practice, because it would make the credentials visible to anyone who has access to the code.

> **Senior Engineer Approach:** A senior engineer would know that there are several different ways to handle environment variables in Next.js. They would be able to explain the pros and cons of each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Different Ways to Set Environment Variables

| Method          | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **`.env` files**| A set of files that can be used to set environment variables for different environments (e.g., `.env.local`, `.env.development`, `.env.production`). |
| **`next.config.js`**| The `env` property in the `next.config.js` file can be used to set environment variables.            |
| **`now.json`**    | If you are deploying your application to Vercel, you can use the `now.json` file to set environment variables. |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use **`.env` files**. This is because it is the most flexible and secure way to handle environment variables.

### Step 3: Code Examples

Here's how we can use `.env` files to set environment variables:

**`.env.local`:**

```
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
```

**`pages/api/my-api.js`:**

```javascript
export default function handler(req, res) {
  const dbHost = process.env.DB_HOST;
  // ...
}
```

### Exposing Environment Variables to the Browser

By default, environment variables are only available on the server. If you want to expose an environment variable to the browser, you need to prefix it with `NEXT_PUBLIC_`.

**`.env.local`:**

```
NEXT_PUBLIC_MY_VAR=myvalue
```

**`pages/my-page.js`:**

```javascript
export default function MyPage() {
  const myVar = process.env.NEXT_PUBLIC_MY_VAR;
  // ...
}
```


---

### Quick Check

**You want to store a secret API key that should not be exposed to the browser. Which of the following would you use?**

   A. A `.env.local` file
   B. A `.env.production` file
   C. A `next.config.js` file
-> D. **Any of the above**

<details>
<summary>See Answer</summary>

Any of these can be used to store a secret API key. However, it is important to make sure that you do not prefix the environment variable with `NEXT_PUBLIC_`, as this will expose it to the browser.

</details>

---

### Question 11: How do you deploy a Next.js application?

**Type:** Practical | **Category:** Deployment

## The Scenario

You are a frontend engineer at a social media company. You have just finished building a new Next.js application, and you need to deploy it to production.

You are not sure which of the different ways to deploy a Next.js application you should use.

## The Challenge

Explain the different ways to deploy a Next.js application. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might just try to deploy the application to a traditional web server. This would work, but it would not be a very scalable or maintainable solution.

> **Senior Engineer Approach:** A senior engineer would know that there are several different ways to deploy a Next.js application. They would be able to explain the pros and cons of each approach and would have a clear recommendation for which one to use in a given situation.

### Step 1: Understand the Different Deployment Options

| Option      | Description                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------- |
| **Vercel**  | The company that created Next.js. Vercel is a platform for deploying and hosting Next.js applications.    |
| **Node.js Server** | You can deploy a Next.js application to a traditional Node.js server.                                 |
| **Static HTML Export** | You can export a Next.js application as a set of static HTML files.                                 |
| **Docker**  | You can deploy a Next.js application as a Docker container.                                             |

### Step 2: Choose the Right Tool for the Job

| Use Case                               | Recommended Option |
| -------------------------------------- | ------------------ |
| A simple application with a few pages  | Vercel             |
| A large application with many components| Vercel or a Node.js server |
| A blog or a documentation website      | Static HTML Export |
| A microservices architecture           | Docker             |

For our use case, we should use **Vercel**. This is because it is the easiest and most scalable way to deploy a Next.js application.

### Step 3: Deploy to Vercel

Here's how we can deploy our application to Vercel:

**1. Create a Vercel account:**

Create a new account on the Vercel website.

**2. Install the Vercel CLI:**

```bash
npm i -g vercel
```

**3. Deploy the application:**

```bash
vercel
```

This command will automatically build and deploy the application to Vercel.


---

### Quick Check

**You are building a blog and you want it to be as fast and SEO-friendly as possible. Which of the following would be the most appropriate?**

   A. Vercel
   B. A Node.js server
-> C. **A static HTML export**
   D. Any of the above

<details>
<summary>See Answer</summary>

A static HTML export is the correct choice for this task. It will pre-render all the pages at build time, which will make the website very fast and SEO-friendly.

</details>

---

### Question 12: What is middleware and how do you use it in Next.js?

**Type:** Practical | **Category:** Middleware

## The Scenario

You are a frontend engineer at a social media company. You are building a new application that needs to have a feature that redirects users from a certain country to a different version of the website.

You could implement this logic in each page, but this would be repetitive and would violate the DRY (Don't Repeat Yourself) principle.

## The Challenge

Explain what middleware is in Next.js and how you would use it to solve this problem. What are the key benefits of using middleware?


### Step 1: Understand What Middleware Is

Middleware is a function that is executed before a request is completed. It can be used to modify the response by rewriting, redirecting, adding headers, or streaming HTML.

### Step 2: Create a Middleware Function

To create a middleware function, you just need to create a `middleware.js` (or `.ts`) file in your `pages` directory.

Here's how we can create a middleware function to redirect users from a certain country:

```javascript
// pages/_middleware.js


export function middleware(req) {
  const { geo } = req
  if (geo.country === 'US') {
    return new Response('Blocked for legal reasons', { status: 451 })
  }
  return NextResponse.next()
}
```

In this example, we check the user's country from the `geo` property of the request. If the user is from the US, we return a `451` status code. Otherwise, we continue to the next middleware or to the page.

### The Benefits of Using Middleware

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Centralization**| You can centralize your logic in a single place, instead of having to repeat it in multiple pages.      |
| **Flexibility**   | Middleware can be used to implement a variety of different features, such as authentication, A/B testing, and internationalization. |


---

### Quick Check

**You want to run a piece of code before every request to a specific set of pages. Which of the following would you use?**

   A. A component
   B. A directive
-> C. **Middleware**
   D. A service

<details>
<summary>See Answer</summary>

Middleware is the correct choice for this task. It allows you to run a piece of code before every request is completed.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

# Complete MongoDB Interview Guide

Master MongoDB with these real-world interview questions covering core concepts, data modeling, and performance optimization. Practice scenarios that mirror actual backend engineering challenges.

**Companies that ask these questions:** Google | Facebook | Netflix | Spotify | Dropbox

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What is the difference between SQL and NoSQL databases? | Conceptual | Core Concepts |
| 2 | How do you do data modeling in MongoDB? | Conceptual | Data Modeling |
| 3 | What are indexes and how do they work in MongoDB? | Conceptual | Indexing |
| 4 | What is the aggregation framework and how do you use it in M... | Conceptual | Aggregation |
| 5 | How do you do replication in MongoDB? | Practical | Replication |
| 6 | How do you do sharding in MongoDB? | Practical | Sharding |
| 7 | What are transactions and how do you use them in MongoDB? | Conceptual | Transactions |
| 8 | What is the `ObjectId` and what is it used for in MongoDB? | Conceptual | Core Concepts |
| 9 | What is GridFS and when would you use it in MongoDB? | Conceptual | Core Concepts |
| 10 | What is the `explain()` method and what is it used for in Mo... | Debugging | Query Optimization |
| 11 | What is the difference between a sparse and a partial index ... | Conceptual | Indexing |
| 12 | What are capped collections and when would you use them in M... | Conceptual | Core Concepts |

---

## What You'll Learn

- Understand the core concepts of MongoDB.
- Master the MongoDB query language.
- Optimize database performance.
- Administer a MongoDB database.
- Work with advanced features like aggregation and indexing.

---

## Interview Questions

### Question 1: What is the difference between SQL and NoSQL databases?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are designing a new service that needs to store a large amount of unstructured data, such as user profiles, posts, and comments.

You are not sure whether to use a SQL database, like PostgreSQL, or a NoSQL database, like MongoDB.

## The Challenge

Explain the difference between SQL and NoSQL databases. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that NoSQL is always better than SQL, or vice versa. They might not be aware of the trade-offs between the two or the specific use cases for which each one is best suited.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between SQL and NoSQL databases. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | SQL                                                              | NoSQL                                                              |
| ------------ | ----------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Data Model** | Relational, data is stored in tables with a fixed schema.         | Non-relational, data can be stored in a variety of different formats, such as documents, key-value pairs, or graphs. |
| **Schema**   | Schema-on-write, the schema is defined before the data is written. | Schema-on-read, the schema is inferred when the data is read.       |
| **Scalability**| Vertically scalable, by adding more resources to a single machine. | Horizontally scalable, by adding more machines to a cluster.        |
| **Use Cases**  | When you need to store structured data and maintain data integrity. | When you need to store unstructured data and scale horizontally.      |

### Step 2: Choose the Right Tool for the Job

For our use case, a **NoSQL database like MongoDB is the best choice**. This is because we are storing a large amount of unstructured data, and we need to be able to scale horizontally.

### Step 3: Data Modeling

Here's how we can model the data in MongoDB:

**User Profile:**

```json
{
  "_id": "user123",
  "name": "John Smith",
  "email": "john.smith@example.com"
}
```

**Post:**

```json
{
  "_id": "post456",
  "user_id": "user123",
  "content": "This is my first post!",
  "comments": [
    {
      "comment_id": "comment789",
      "user_id": "user456",
      "content": "Nice post!"
    }
  ]
}
```

By using a document-oriented database like MongoDB, we can store the data in a way that is more natural and intuitive than a relational database.


---

### Quick Check

**You are building a financial application that requires strong data consistency and a fixed schema. Which of the following would be the most appropriate?**

-> A. **SQL**
   B. NoSQL
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

A SQL database is the correct choice for this task. It provides strong data consistency and a fixed schema, which are important for financial applications.

</details>

---

### Question 2: How do you do data modeling in MongoDB?

**Type:** Conceptual | **Category:** Data Modeling

## The Scenario

You are a backend engineer at an e-commerce company. You are designing a new service that will store information about products and their reviews.

You have decided to use MongoDB for this service, but you are not sure how to model the data.

## The Challenge

Explain how you would do data modeling in MongoDB. What are the different data modeling patterns that you would use, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might try to model the data in the same way that they would in a relational database. This would be a mistake, because MongoDB is a document-oriented database, and it has a different data modeling paradigm.

> **Senior Engineer Approach:** A senior engineer would know that data modeling in MongoDB is all about embedding and linking. They would be able to explain the different data modeling patterns and would have a clear plan for how to model the data for this use case.

### Step 1: Understand the Key Concepts

| Concept      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **Embedding**| Storing related data in a single document.                                                              |
| **Linking**  | Storing a reference to another document.                                                                |

### Step 2: Choose the Right Pattern for the Job

| Use Case                               | Recommended Pattern |
| -------------------------------------- | ------------------- |
| **One-to-one relationships**           | Embedding           |
| **One-to-many relationships**          | Embedding or linking, depending on the use case. |
| **Many-to-many relationships**         | Linking             |

For our use case, we have a one-to-many relationship between products and reviews. A product can have many reviews, and a review belongs to a single product.

We could model this in two ways:

1.  **Embed the reviews in the product document:** This would be a good choice if we always want to retrieve the reviews along with the product.
2.  **Link the reviews to the product document:** This would be a good choice if we want to be able to retrieve the reviews independently of the product.

In this case, we will choose to **embed the reviews in the product document**. This is because we will almost always want to retrieve the reviews along with the product.

### Step 3: Code Examples

Here's how we can model the data with embedded reviews:

```json
{
  "_id": "product123",
  "name": "My Awesome Product",
  "reviews": [
    {
      "review_id": "review456",
      "user_id": "user123",
      "content": "This product is great!"
    },
    {
      "review_id": "review789",
      "user_id": "user456",
      "content": "This product is not so great."
    }
  ]
}
```


---

### Quick Check

**You are building a social media application and need to model the relationship between users and their followers. Which of the following would be the most appropriate?**

   A. Embedding
-> B. **Linking**
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

Linking is the correct choice for this task. A user can have many followers, and a follower can be following many users. This is a many-to-many relationship, which is best modeled with linking.

</details>

---

### Question 3: What are indexes and how do they work in MongoDB?

**Type:** Conceptual | **Category:** Indexing

## The Scenario

You are a backend engineer at an e-commerce company. You are responsible for a service that is experiencing performance issues. The service is slow to respond to requests, and you have identified that the bottleneck is a slow database query.

The query is searching for documents in a large collection, and it is taking several seconds to execute.

## The Challenge

Explain what indexes are in MongoDB and how they work. What are the different types of indexes, and how would you use them to optimize the performance of the slow query?


> **Common Mistake:** A junior engineer might not be aware of indexes. They might just create a collection without any indexes, which would lead to poor performance for large collections.

> **Senior Engineer Approach:** A senior engineer would know that indexes are a critical part of database performance. They would be able to explain what indexes are and how they work. They would also be able to explain the different types of indexes and would have a clear plan for how to use them to optimize the performance of their queries.

### Step 1: Understand What Indexes Are

An index is a data structure that is used to speed up the process of finding documents in a collection. It works by creating a copy of the indexed field(s) and storing them in a sorted order. This allows the database to quickly find the documents that match a query without having to scan the entire collection.

### Step 2: The Different Types of Indexes

| Index Type        | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Single Field**  | An index on a single field.                                                                             |
| **Compound**      | An index on multiple fields.                                                                            |
| **Multikey**      | An index on an array field.                                                                             |
| **Text**          | An index on a string field that can be used for full-text search.                                       |
| **Geospatial**    | An index on a field that contains geospatial data.                                                      |

### Step 3: Choose the Right Tool for the Job

| Use Case                               | Recommended Index Type |
| -------------------------------------- | ---------------------- |
| Most use cases                         | Single Field or Compound |
| Indexing an array field                | Multikey               |
| Full-text search                       | Text                   |
| Geospatial queries                     | Geospatial             |

### Step 4: Code Examples

Here's how we can create an index in MongoDB:

```javascript
db.my_collection.createIndex({ my_field: 1 })
```

### The Trade-offs of Using Indexes

| Pros                               | Cons                                                              |
| ---------------------------------- | ----------------------------------------------------------------- |
| Speeds up `find` queries.          | Slows down `insert`, `update`, and `remove` queries.              |
|                                    | Takes up disk space.                                              |

Because of these trade-offs, you should only create indexes on the fields that you frequently use in your queries.


---

### Quick Check

**You want to create an index on an array field that contains a list of tags. Which of the following would be the most appropriate?**

   A. Single Field
   B. Compound
-> C. **Multikey**
   D. Text

<details>
<summary>See Answer</summary>

A multikey index is the correct choice for this task. It is designed for indexing array fields, and it will allow you to efficiently query for documents that contain a specific tag.

</details>

---

### Question 4: What is the aggregation framework and how do you use it in MongoDB?

**Type:** Conceptual | **Category:** Aggregation

## The Scenario

You are a data analyst at an e-commerce company. You are trying to write a query to find the total sales for each category.

You could do this by fetching all the data from the database and then processing it in your application, but you know that this would be inefficient.

## The Challenge

Explain what the aggregation framework is in MongoDB and how you would use it to solve this problem. What are the key stages of an aggregation pipeline?


> **Common Mistake:** A junior engineer might not be aware of the aggregation framework. They might try to solve this problem by fetching all the data from the database and then processing it in their application, which would be inefficient.

> **Senior Engineer Approach:** A senior engineer would know that the aggregation framework is the perfect tool for this job. They would be able to explain what the aggregation framework is and how to use it to write concise and efficient queries for a variety of different use cases.

### Step 1: Understand What the Aggregation Framework Is

The aggregation framework is a tool for performing data analysis on a collection of documents. It works by processing the documents through a pipeline of stages.

### Step 2: The Key Stages of an Aggregation Pipeline

| Stage      | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **`$match`** | Filters the documents to only include the ones that match a given criteria.                             |
| **`$group`** | Groups the documents by a given key and performs an aggregation function on each group.                  |
| **`$sort`**  | Sorts the documents by a given key.                                                                     |
| **`$project`**| Reshapes the documents by adding, removing, or renaming fields.                                          |
| **`$limit`** | Limits the number of documents that are passed to the next stage.                                       |
| **`$skip`**  | Skips a specified number of documents.                                                                  |

### Step 3: Solve the Problem

Here's how we can use the aggregation framework to find the total sales for each category:

```javascript
db.products.aggregate([
    {
        $group: {
            _id: "$category",
            total_sales: { $sum: "$sales" }
        }
    },
    {
        $sort: {
            total_sales: -1
        }
    }
])
```

In this example, we use the `$group` stage to group the products by category and to calculate the total sales for each category. We then use the `$sort` stage to sort the results by total sales in descending order.


---

### Quick Check

**You want to find the average price of all the products in each category. Which of the following would you use?**

   A. `$sum`
-> B. **`$avg`**
   C. `$min`
   D. `$max`

<details>
<summary>See Answer</summary>

`$avg` is the correct choice for this task. It is an accumulator operator that can be used in the `$group` stage to calculate the average of a numeric field.

</details>

---

### Question 5: How do you do replication in MongoDB?

**Type:** Practical | **Category:** Replication

## The Scenario

You are a database administrator at a social media company. You are responsible for a MongoDB database that is critical to the business.

You need to set up replication to a secondary database to provide high availability and disaster recovery.

## The Challenge

Explain how you would set up replication in MongoDB. What is a replica set, and what are the different types of nodes in a replica set?


> **Common Mistake:** A junior engineer might not be aware of replication. They might try to solve this problem by just taking periodic backups of the database, which would not provide high availability.

> **Senior Engineer Approach:** A senior engineer would know that replication is a critical part of database administration. They would be able to explain what a replica set is and would have a clear plan for how to set up replication for a production database.

### Step 1: Understand What a Replica Set Is

A replica set is a group of MongoDB servers that maintain the same data set. A replica set provides redundancy and high availability.

### Step 2: The Different Types of Nodes

| Node Type       | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **Primary**     | The primary node is the only node that can accept write operations.                                     |
| **Secondary**   | The secondary nodes replicate the data from the primary node. They can be used for read operations.      |
| **Arbiter**     | An arbiter does not store any data, but it participates in elections to choose a new primary.             |

### Step 3: Set Up a Replica Set

Here's how we can set up a simple replica set with one primary and two secondaries:

**1. Start the MongoDB servers:**

Start three MongoDB servers on different machines.

**2. Initiate the replica set:**

Connect to one of the servers and initiate the replica set.

```javascript
rs.initiate({
    _id: "myReplicaSet",
    members: [
        { _id: 0, host: "host1:27017" },
        { _id: 1, host: "host2:27017" },
        { _id: 2, host: "host3:27017" }
    ]
})
```

**3. Check the status of the replica set:**

You can check the status of the replica set by running the `rs.status()` command.

### Failover

If the primary node goes down, the remaining nodes will hold an election to choose a new primary. This process is automatic and ensures that the database is always available.


---

### Quick Check

**You have a three-node replica set and you want to be able to perform reads from the secondary nodes. What should you do?**

   A. Set the read preference to `primary`.
   B. Set the read preference to `secondary`.
   C. Set the read preference to `nearest`.
-> D. **All of the above**

<details>
<summary>See Answer</summary>

You can use the read preference to control which nodes are used for read operations. By default, the read preference is set to `primary`, which means that all read operations are sent to the primary node. You can change the read preference to `secondary` to perform reads from the secondary nodes, or to `nearest` to perform reads from the nearest node.

</details>

---

### Question 6: How do you do sharding in MongoDB?

**Type:** Practical | **Category:** Sharding

## The Scenario

You are a database administrator at a social media company. You are responsible for a MongoDB database that is growing very quickly. The database is starting to experience performance issues, and you have identified that the bottleneck is the single server that is hosting the database.

You need to find a way to scale the database horizontally to handle the increasing load.

## The Challenge

Explain how you would do sharding in MongoDB. What is a sharded cluster, and what are the different components of a sharded cluster?


> **Common Mistake:** A junior engineer might not be aware of sharding. They might try to solve this problem by just adding more resources to the single server, which would not be a very scalable solution.

> **Senior Engineer Approach:** A senior engineer would know that sharding is a critical part of database administration. They would be able to explain what a sharded cluster is and would have a clear plan for how to set up sharding for a production database.

### Step 1: Understand What a Sharded Cluster Is

A sharded cluster is a group of MongoDB servers that work together to store and process data. A sharded cluster provides horizontal scalability by distributing the data across multiple servers.

### Step 2: The Different Components of a Sharded Cluster

| Component       | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **Shard**       | Each shard is a separate MongoDB server that stores a subset of the data.                               |
| **Config Servers**| The config servers store the metadata for the sharded cluster, such as the location of each shard.      |
| **Query Routers**| The query routers (mongos) are responsible for routing queries to the correct shard.                    |

### Step 3: Set Up a Sharded Cluster

Here's how we can set up a simple sharded cluster with two shards:

**1. Start the config servers:**

Start three config servers on different machines.

**2. Start the query routers:**

Start one or more query routers.

**3. Add the shards:**

Connect to one of the query routers and add the shards to the cluster.

```javascript
sh.addShard("shard1.example.com:27017")
sh.addShard("shard2.example.com:27017")
```

**4. Enable sharding for a database:**

```javascript
sh.enableSharding("mydb")
```

**5. Shard a collection:**

```javascript
sh.shardCollection("mydb.mycollection", { my_key: 1 })
```

### Shard Keys

The shard key is the field that is used to distribute the data across the shards. It is important to choose a good shard key, as it will have a big impact on the performance of the sharded cluster.


---

### Quick Check

**You are designing a sharded cluster for a new application. Which of the following would be the most important consideration when choosing a shard key?**

   A. The cardinality of the shard key.
   B. The frequency of queries on the shard key.
   C. The write distribution of the shard key.
-> D. **All of the above**

<details>
<summary>See Answer</summary>

All of these are important considerations when choosing a shard key. A good shard key should have a high cardinality, a high frequency of queries, and a good write distribution.

</details>

---

### Question 7: What are transactions and how do you use them in MongoDB?

**Type:** Conceptual | **Category:** Transactions

## The Scenario

You are a backend engineer at a fintech company. You are writing a new service that needs to transfer money from one account to another.

The transfer consists of two separate operations: a `updateOne` to the sender's account to debit the money, and a `updateOne` to the receiver's account to credit the money.

You need to make sure that both operations are executed successfully, or that neither of them are.

## The Challenge

Explain what transactions are in MongoDB and how you would use them to solve this problem. What are the key properties of a transaction (ACID)?


### Step 1: Understand What Transactions Are

A transaction is a sequence of one or more operations that are executed as a single logical unit.

### Step 2: The ACID Properties

MongoDB transactions are ACID compliant.

| Property       | Description                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------- |
| **Atomicity**  | All the operations in a transaction are executed successfully, or none of them are.                   |
| **Consistency**| A transaction brings the database from one valid state to another.                                      |
| **Isolation**  | The intermediate state of a transaction is not visible to other transactions.                            |
| **Durability** | Once a transaction has been committed, it will remain committed even in the event of a power failure.      |

### Step 3: Use a Transaction

Here's how we can use a transaction to transfer money from one account to another:

```javascript
const session = db.getMongo().startSession();

session.startTransaction();

try {
    session.getDatabase("mydb").getCollection("accounts").updateOne(
        { _id: 1 },
        { $inc: { balance: -100 } }
    );
    session.getDatabase("mydb").getCollection("accounts").updateOne(
        { _id: 2 },
        { $inc: { balance: 100 } }
    );
} catch (error) {
    session.abortTransaction();
    throw error;
}

session.commitTransaction();
session.endSession();
```

In this example, we start a new session and a new transaction. We then execute the two `updateOne` statements. If both statements are successful, we commit the transaction. If either statement fails, we abort the transaction.


---

### Quick Check

**You are in the middle of a transaction and you want to undo all the changes that you have made so far. Which of the following would you use?**

   A. `commitTransaction()`
-> B. **`abortTransaction()`**
   C. `endSession()`
   D. None of the above

<details>
<summary>See Answer</summary>

`abortTransaction()` is the correct choice for this task. It will undo all the changes that you have made in the current transaction.

</details>

---

### Question 8: What is the `ObjectId` and what is it used for in MongoDB?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are designing a new service that will store user-generated content, such as posts and comments.

You need to choose a primary key for your documents, and you are considering using the `ObjectId` data type.

## The Challenge

Explain what the `ObjectId` is in MongoDB and why it is a good choice for a primary key. What are the different parts of an `ObjectId`, and what do they represent?


> **Common Mistake:** A junior engineer might not be aware of the `ObjectId` data type. They might try to use an auto-incrementing integer as the primary key, which is not a good choice for a distributed database like MongoDB.

> **Senior Engineer Approach:** A senior engineer would know that the `ObjectId` is the default primary key in MongoDB. They would be able to explain what an `ObjectId` is and why it is a good choice for a primary key. They would also be able to explain the different parts of an `ObjectId` and what they represent.

### Step 1: Understand What an `ObjectId` Is

An `ObjectId` is a 12-byte BSON type that is the default primary key for documents in MongoDB.

### Step 2: The Different Parts of an `ObjectId`

An `ObjectId` is made up of the following parts:

| Bytes      | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **4**      | A timestamp, representing the `ObjectId`'s creation time.                                               |
| **3**      | A machine identifier.                                                                                   |
| **2**      | A process identifier.                                                                                   |
| **3**      | A counter, starting with a random value.                                                                |

### The Benefits of Using an `ObjectId`

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Uniqueness**    | `ObjectId`s are unique across a sharded cluster.                                                        |
| **Sortability**   | `ObjectId`s are sortable by creation time.                                                              |
| **Performance**   | `ObjectId`s are small and efficient to index.                                                           |

### When to Use a Custom Primary Key

In some cases, you might want to use a custom primary key instead of an `ObjectId`. For example, you might use a user's email address as the primary key for a `users` collection.

However, in most cases, the `ObjectId` is a good choice for a primary key.


---

### Quick Check

**You want to be able to sort your documents by creation time. Which of the following would be the most appropriate?**

   A. A custom timestamp field
-> B. **The `ObjectId`**
   C. Either is fine
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

The `ObjectId` is the correct choice for this task. The first 4 bytes of an `ObjectId` are a timestamp, so you can sort your documents by `ObjectId` to sort them by creation time.

</details>

---

### Question 9: What is GridFS and when would you use it in MongoDB?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to store large files, such as images and videos.

You are considering using either the file system or GridFS to store the files.

## The Challenge

Explain what GridFS is in MongoDB and why it is a better choice than the file system for storing large files. What are the key benefits of using GridFS?


> **Common Mistake:** A junior engineer might try to store the large files in a `BSON` document. This would not work, because `BSON` documents have a size limit of 16MB. They might not be aware of GridFS, which is the correct tool for this job.

> **Senior Engineer Approach:** A senior engineer would know that GridFS is the perfect tool for this job. They would be able to explain what GridFS is and how to use it to store and retrieve large files. They would also be able to explain the benefits of using GridFS over the file system.

### Step 1: Understand What GridFS Is

GridFS is a specification for storing and retrieving large files in MongoDB. It works by dividing a large file into smaller chunks and storing each chunk as a separate document.

### Step 2: The `fs.files` and `fs.chunks` Collections

GridFS uses two collections to store the files:

| Collection      | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **`fs.files`**  | Stores the metadata for the files, such as the file name, the content type, and the length.             |
| **`fs.chunks`** | Stores the chunks of the files.                                                                         |

### The Benefits of Using GridFS

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Scalability**   | GridFS can be used to store files that are larger than the 16MB BSON document size limit.               |
| **Replication**   | GridFS files are automatically replicated across the nodes in a replica set.                            |
| **Sharding**      | GridFS files are automatically sharded across the shards in a sharded cluster.                          |

### When to Use GridFS

You should use GridFS when you need to store files that are larger than the 16MB BSON document size limit.

You should not use GridFS if you need to perform atomic updates on the content of a file.


---

### Quick Check

**You are building a service that needs to store and retrieve large video files. Which of the following would be the most appropriate?**

   A. A `BSON` document
   B. The file system
-> C. **GridFS**
   D. None of the above

<details>
<summary>See Answer</summary>

GridFS is the correct choice for this task. It is designed for storing and retrieving large files, and it provides scalability, replication, and sharding.

</details>

---

### Question 10: What is the `explain()` method and what is it used for in MongoDB?

**Type:** Debugging | **Category:** Query Optimization

## The Scenario

You are a backend engineer at an e-commerce company. You are responsible for a service that is experiencing performance issues. The service is slow to respond to requests, and you have identified that the bottleneck is a slow database query.

## The Challenge

Explain what the `explain()` method is in MongoDB and how you would use it to optimize the slow database query. What are the key things to look for in the output of the `explain()` method?


> **Common Mistake:** A junior engineer might not be aware of the `explain()` method. They might try to solve the problem by just adding more indexes, which might not be the most effective solution. They might not be aware of the other tools that can be used to analyze the query plan.

> **Senior Engineer Approach:** A senior engineer would know that the `explain()` method is a critical part of query optimization. They would be able to explain how to use the `explain()` method to analyze the query plan, and they would have a clear plan for how to use this information to optimize the query.

### Step 1: Understand What the `explain()` Method Is

The `explain()` method is a tool that can be used to analyze the performance of a query. It returns a document that describes the query plan, which is the sequence of operations that the database will perform to execute the query.

### Step 2: Use the `explain()` Method

Here's how we can use the `explain()` method to analyze a query:

```javascript
db.my_collection.find({ my_field: "my_value" }).explain("executionStats")
```

The `executionStats` argument tells the `explain()` method to execute the query and to include the execution statistics in the output.

### Step 3: Analyze the Output

The output of the `explain()` method is a JSON document that contains a wealth of information about the query plan.

| Key                          | Description                                                                                             |
| ---------------------------- | ------------------------------------------------------------------------------------------------------- |
| **`queryPlanner`**           | Information about the query plan that was chosen by the query optimizer.                                  |
| **`executionStats`**         | Statistics about the execution of the query, such as the number of documents scanned and the execution time. |
| **`winningPlan.stage`**      | The winning query plan. Look for `COLLSCAN` which indicates a full collection scan.                       |
| **`executionStats.nReturned`** | The number of documents that were returned by the query.                                                |
| **`executionStats.totalKeysExamined`** | The number of index keys that were examined.                                                    |
| **`executionStats.totalDocsExamined`** | The number of documents that were examined.                                                   |

### Step 4: Identify and Fix the Bottleneck

By analyzing the output of the `explain()` method, you can identify the bottleneck in your query and take steps to fix it.

For example, if you see that the `totalDocsExamined` is much larger than the `nReturned`, it means that the database is scanning a lot of documents that are not being returned by the query. This is a sign that you need to add an index to the collection.


---

### Quick Check

**You are looking at the output of the `explain()` method and you see that the `winningPlan.stage` is `COLLSCAN`. What does this mean?**

   A. The query is using an index.
   B. The query is not using an index.
   C. The query is performing a full collection scan.
-> D. **Both b and c.**

<details>
<summary>See Answer</summary>

`COLLSCAN` means that the query is performing a full collection scan, which means that it is not using an index. This is usually a sign that you need to add an index to the collection.

</details>

---

### Question 11: What is the difference between a sparse and a partial index in MongoDB?

**Type:** Conceptual | **Category:** Indexing

## The Scenario

You are a backend engineer at a social media company. You are designing a new service that will store user profiles. Some users have a `location` field, but others do not.

You want to create an index on the `location` field to speed up queries that search for users by location. However, you do not want the index to include the users who do not have a `location` field.

## The Challenge

Explain the difference between a sparse and a partial index in MongoDB. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might not be aware of sparse or partial indexes. They might just create a regular index on the `location` field, which would include all the documents in the collection, even the ones that do not have a `location` field.

> **Senior Engineer Approach:** A senior engineer would know that a sparse index is the perfect tool for this job. They would be able to explain the difference between a sparse and a partial index, and they would have a clear plan for how to use them to solve this problem.

### Step 1: Understand the Key Differences

| Feature      | Sparse Index                                                              | Partial Index                                                              |
| ------------ | ------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Inclusion**| Only includes documents that have the indexed field.                      | Only includes documents that match a given filter expression.              |
| **Syntax**   | `db.my_collection.createIndex({ my_field: 1 }, { sparse: true })`         | `db.my_collection.createIndex({ my_field: 1 }, { partialFilterExpression: { my_field: { $exists: true } } })` |
| **Use Cases**  | When you want to index a field that only exists in some of the documents. | When you want to index a subset of the documents in a collection.          |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use a **sparse index**. This is because we want to index the `location` field, but we only want the index to include the documents that have a `location` field.

A partial index would also work for this use case, but a sparse index is simpler to create and is more efficient for this specific scenario.


---

### Quick Check

**You want to create an index on a field that only contains a certain value. Which of the following would be the most appropriate?**

   A. A sparse index
-> B. **A partial index**
   C. A compound index
   D. A multikey index

<details>
<summary>See Answer</summary>

A partial index is the correct choice for this task. It allows you to create an index on a subset of the documents in a collection, based on a filter expression.

</details>

---

### Question 12: What are capped collections and when would you use them in MongoDB?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to store a log of the most recent user activity.

You need to find a way to store the logs in a way that is efficient and that automatically removes old logs.

## The Challenge

Explain what capped collections are in MongoDB and how you would use them to solve this problem. What are the key benefits of using capped collections?


> **Common Mistake:** A junior engineer might try to solve this problem by using a regular collection and then writing a separate script to periodically remove old logs. This would be a complex and inefficient solution.

> **Senior Engineer Approach:** A senior engineer would know that a capped collection is the perfect tool for this job. They would be able to explain what a capped collection is and how to use it to automatically remove old logs.

### Step 1: Understand What Capped Collections Are

A capped collection is a fixed-size collection that automatically overwrites its oldest entries when it reaches its maximum size.

### Step 2: The Key Features of Capped Collections

| Feature      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **Size**     | Capped collections have a fixed size, which you specify when you create the collection.                  |
| **Ordering** | Capped collections maintain the insertion order of the documents.                                       |
| **TTL**      | Capped collections do not have a TTL (Time To Live) index.                                              |
| **Updates**  | You cannot update documents in a capped collection if the update would cause the document to grow in size. |
| **Deletes**  | You cannot delete documents from a capped collection.                                                   |

### Step 3: Use a Capped Collection

Here's how we can create a capped collection to store our logs:

```javascript
db.createCollection("my_logs", { capped: true, size: 100000 })
```

In this example, we create a new capped collection called `my_logs` with a maximum size of 100,000 bytes.

### The Benefits of Using Capped Collections

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Performance**   | Capped collections are very performant, because they are designed for high-throughput logging.           |
| **Simplicity**    | Capped collections are easy to use, and they automatically handle the removal of old logs.              |


---

### Quick Check

**You are building a service that needs to store a log of the most recent user activity. Which of the following would be the most appropriate?**

   A. A regular collection
-> B. **A capped collection**
   C. A time-series collection
   D. None of the above

<details>
<summary>See Answer</summary>

A capped collection is the correct choice for this task. It is designed for high-throughput logging, and it automatically removes old logs when it reaches its maximum size.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

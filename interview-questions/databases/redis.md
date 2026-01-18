# Complete Redis Interview Guide

Master Redis with these real-world interview questions covering core concepts, data structures, and performance optimization. Practice scenarios that mirror actual backend engineering challenges.

**Companies that ask these questions:** Twitter | GitHub | Stack Overflow | Pinterest | Craigslist

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | What are the different data structures in Redis and when wou... | Conceptual | Data Structures |
| 2 | How do you do persistence in Redis? | Practical | Persistence |
| 3 | How do you do replication in Redis? | Practical | Replication |
| 4 | How do you do sharding in Redis? | Practical | Sharding |
| 5 | What are transactions and how do you use them in Redis? | Conceptual | Transactions |
| 6 | What is Lua scripting and how do you use it in Redis? | Practical | Lua scripting |
| 7 | What is Redis Sentinel and how do you use it for high availa... | Practical | High Availability |
| 8 | What is Redis Cluster and how do you use it for scalability? | Practical | Scalability |
| 9 | What is the difference between `DEL` and `UNLINK` in Redis? | Conceptual | Core Concepts |
| 10 | What are Redis Streams and what are they used for? | Conceptual | Data Structures |
| 11 | What are bitmaps and hyperloglogs and what are they used for... | Conceptual | Data Structures |
| 12 | How do you use Redis as a cache? | Practical | Caching |

---

## What You'll Learn

- Understand the core concepts of Redis.
- Master the Redis data structures.
- Optimize database performance.
- Administer a Redis database.
- Work with advanced features like Lua scripting and transactions.

---

## Interview Questions

### Question 1: What are the different data structures in Redis and when would you use them?

**Type:** Conceptual | **Category:** Data Structures

## The Scenario

You are a backend engineer at a social media company. You are designing a new service that needs to store a variety of different types of data, such as user profiles, posts, and timelines.

You have decided to use Redis for this service, but you are not sure which data structures to use for each type of data.

## The Challenge

Explain the different data structures in Redis and when you would use them. What are the pros and cons of each data structure, and which one would you choose for each of the different types of data in this use case?


> **Common Mistake:** A junior engineer might try to store all the data as strings. This would work, but it would be very inefficient and would not take advantage of the powerful data structures that are available in Redis.

> **Senior Engineer Approach:** A senior engineer would know that Redis has a variety of different data structures that are each designed for a specific use case. They would be able to explain the pros and cons of each data structure and would have a clear plan for how to use them to model the data for this use case.

### Step 1: Understand the Key Data Structures

| Data Structure | Description                                                                                             | Use Cases                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Strings**    | The simplest data structure in Redis. Can store any type of data, such as a string, an integer, or a float. | Caching, counters.                                                                                     |
| **Lists**      | A list of strings, sorted by insertion order.                                                           | Queues, timelines.                                                                                      |
| **Sets**       | An unordered collection of unique strings.                                                              | Tags, unique visitors.                                                                                  |
| **Hashes**     | A collection of key-value pairs.                                                                        | User profiles, product catalogs.                                                                        |
| **Sorted Sets**| A collection of unique strings that are sorted by a score.                                             | Leaderboards, rate limiting.                                                                            |

### Step 2: Choose the Right Tool for the Job

| Data Type       | Recommended Data Structure |
| --------------- | -------------------------- |
| **User profiles** | Hashes                     |
| **Posts**       | Hashes                     |
| **Timelines**   | Lists or Sorted Sets       |

### Step 3: Code Examples

Here are some code examples that show how to use some of these data structures:

**Hashes:**

```
HSET user:123 name "John Smith"
HSET user:123 email "john.smith@example.com"
```

**Lists:**

```
LPUSH timeline:123 "post:456"
LPUSH timeline:123 "post:789"
```

**Sorted Sets:**

```
ZADD leaderboard 100 "user:123"
ZADD leaderboard 200 "user:456"
```


---

### Quick Check

**You are building a real-time leaderboard for a gaming application. Which of the following would be the most appropriate?**

   A. Lists
   B. Sets
   C. Hashes
-> D. **Sorted Sets**

<details>
<summary>See Answer</summary>

A sorted set is the correct choice for this task. It is a collection of unique strings that are sorted by a score, which makes it perfect for building a leaderboard.

</details>

---

### Question 2: How do you do persistence in Redis?

**Type:** Practical | **Category:** Persistence

## The Scenario

You are a database administrator at a social media company. You are responsible for a Redis database that is critical to the business.

You need to make sure that the data in the database is not lost in the event of a server restart or a power failure.

## The Challenge

Explain how you would do persistence in Redis. What are the different persistence options, and what are the trade-offs between them?


> **Common Mistake:** A junior engineer might not be aware of persistence in Redis. They might just use Redis as an in-memory cache, which would not be a very robust solution for a production database.

> **Senior Engineer Approach:** A senior engineer would know that persistence is a critical part of database administration. They would be able to explain the different persistence options and would have a clear plan for how to set up persistence for a production database.

### Step 1: Understand the Different Persistence Options

| Option      | Description                                                                                             | Pros                                                              | Cons                                                               |
| ----------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------ |
| **RDB**     | A point-in-time snapshot of the database.                                                               | Fast and easy to use.                                             | Can result in data loss if the server crashes between snapshots.   |
| **AOF**     | An append-only log of all the write operations that are performed on the database.                      | More durable than RDB, can be configured to fsync on every write. | Can be slower than RDB, and the AOF file can grow to be very large. |
| **RDB + AOF** | A combination of RDB and AOF.                                                                           | The best of both worlds: fast and durable.                         | More complex to set up than RDB or AOF alone.                      |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use **RDB + AOF**. This will give us the best of both worlds: the speed of RDB and the durability of AOF.

### Step 3: Configure Persistence

Here's how we can configure persistence in the `redis.conf` file:

```
# RDB
save 900 1
save 300 10
save 60 10000

# AOF
appendonly yes
appendfsync everysec
```

In this example, we have configured Redis to create an RDB snapshot every 15 minutes if at least 1 key has changed, every 5 minutes if at least 10 keys have changed, and every 1 minute if at least 10000 keys have changed. We have also configured Redis to use AOF persistence and to fsync the AOF file every second.


---

### Quick Check

**You need to be able to restore your database to any point in time. Which of the following would you use?**

   A. RDB
-> B. **AOF**
   C. RDB + AOF
   D. None of the above

<details>
<summary>See Answer</summary>

AOF is the correct choice for this task. It is an append-only log of all the write operations, which means that you can replay the log to restore the database to any point in time.

</details>

---

### Question 3: How do you do replication in Redis?

**Type:** Practical | **Category:** Replication

## The Scenario

You are a database administrator at a social media company. You are responsible for a Redis database that is critical to the business.

You need to set up replication to a secondary database to provide high availability and disaster recovery.

## The Challenge

Explain how you would set up replication in Redis. What is a replica set, and what are the different types of nodes in a replica set?


> **Common Mistake:** A junior engineer might not be aware of replication. They might try to solve this problem by just taking periodic backups of the database, which would not provide high availability.

> **Senior Engineer Approach:** A senior engineer would know that replication is a critical part of database administration. They would be able to explain what a replica set is and would have a clear plan for how to set up replication for a production database.

### Step 1: Understand What a Replica Set Is

A replica set is a group of Redis servers that maintain the same data set. A replica set provides redundancy and high availability.

### Step 2: The Different Types of Nodes

| Node Type       | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **Master**      | The master node is the only node that can accept write operations.                                      |
| **Slave**       | The slave nodes replicate the data from the master node. They can be used for read operations.         |

### Step 3: Set Up a Replica Set

Here's how we can set up a simple replica set with one master and two slaves:

**1. Configure the master server:**

Edit the `redis.conf` file and set a password for the master server.

```
requirepass mypassword
```

**2. Configure the slave servers:**

Edit the `redis.conf` file on each slave server and set the following parameters:

```
replicaof <master_ip> <master_port>
masterauth mypassword
```

**3. Start the servers:**

Start the master server and then start the slave servers.

**4. Check the status of the replica set:**

You can check the status of the replica set by running the `INFO replication` command on the master server.

### Failover

If the master node goes down, you will need to manually promote one of the slave nodes to be the new master. You can do this by running the `REPLICAOF NO ONE` command on the slave node.

For automatic failover, you can use Redis Sentinel.


---

### Quick Check

**You have a three-node replica set and you want to be able to perform reads from the slave nodes. What should you do?**

-> A. **Nothing, this is the default behavior.**
   B. Set the `slave-read-only` parameter to `no`.
   C. Use the `READONLY` command.
   D. It's not possible to perform reads from the slave nodes.

<details>
<summary>See Answer</summary>

By default, slave nodes are read-only. You can connect to a slave node and perform read operations, but you cannot perform write operations.

</details>

---

### Question 4: How do you do sharding in Redis?

**Type:** Practical | **Category:** Sharding

## The Scenario

You are a database administrator at a social media company. You are responsible for a Redis database that is growing very quickly. The database is starting to experience performance issues, and you have identified that the bottleneck is the single server that is hosting the database.

You need to find a way to scale the database horizontally to handle the increasing load.

## The Challenge

Explain how you would do sharding in Redis. What is a sharded cluster, and what are the different components of a sharded cluster?


> **Common Mistake:** A junior engineer might not be aware of sharding. They might try to solve this problem by just adding more resources to the single server, which would not be a very scalable solution.

> **Senior Engineer Approach:** A senior engineer would know that sharding is a critical part of database administration. They would be able to explain what a sharded cluster is and would have a clear plan for how to set up sharding for a production database.

### Step 1: Understand What a Sharded Cluster Is

A sharded cluster is a group of Redis servers that work together to store and process data. A sharded cluster provides horizontal scalability by distributing the data across multiple servers.

### Step 2: The Different Components of a Sharded Cluster

| Component       | Description                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **Shard**       | Each shard is a separate Redis server that stores a subset of the data.                               | 
| **Redis Cluster Bus**| A communication bus that is used by the nodes in the cluster to exchange information about the state of the cluster. |

### Step 3: Set Up a Sharded Cluster

Here's how we can set up a simple sharded cluster with three masters and three slaves:

**1. Create the `redis.conf` files:**

Create a separate `redis.conf` file for each node in the cluster.

```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

**2. Start the servers:**

Start six Redis servers on different machines, each with its own `redis.conf` file.

**3. Create the cluster:**

Use the `redis-cli` tool to create the cluster.

```bash
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
--cluster-replicas 1
```

### Hash Slots

Redis Cluster uses a concept called hash slots to distribute the data across the shards. There are 16384 hash slots in a Redis Cluster.

When you set a key, Redis calculates a hash of the key and then uses the result to determine which hash slot the key belongs to. It then sends the key to the shard that is responsible for that hash slot.


---

### Quick Check

**You are designing a sharded cluster for a new application. Which of the following would be the most important consideration when choosing a sharding strategy?**

   A. The number of shards.
   B. The number of replicas.
   C. The way that the data is distributed across the shards.
-> D. **All of the above**

<details>
<summary>See Answer</summary>

All of these are important considerations when choosing a sharding strategy. A good sharding strategy should have a sufficient number of shards and replicas, and it should distribute the data evenly across the shards.

</details>

---

### Question 5: What are transactions and how do you use them in Redis?

**Type:** Conceptual | **Category:** Transactions

## The Scenario

You are a backend engineer at a fintech company. You are writing a new service that needs to transfer money from one account to another.

The transfer consists of two separate operations: a `DECRBY` to the sender's account to debit the money, and a `INCRBY` to the receiver's account to credit the money.

You need to make sure that both operations are executed successfully, or that neither of them are.

## The Challenge

Explain what transactions are in Redis and how you would use them to solve this problem. What are the key commands that you would use to create a transaction?


### Step 1: Understand What Transactions Are

A transaction is a sequence of one or more commands that are executed as a single atomic unit.

### Step 2: The Key Commands

| Command    | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **`MULTI`**  | Starts a new transaction.                                                                               |
| **`EXEC`**   | Executes all the commands in the transaction.                                                           |
| **`DISCARD`**| Discards all the commands in the transaction.                                                           |
| **`WATCH`**  | Watches one or more keys to detect changes to them.                                                     |

### Step 3: Use a Transaction

Here's how we can use a transaction to transfer money from one account to another:

```
WATCH account:1 account:2
balance1 = GET account:1
balance2 = GET account:2
if balance1 >= 100:
    MULTI
    DECRBY account:1 100
    INCRBY account:2 100
    EXEC
else:
    UNWATCH
```

In this example, we use the `WATCH` command to watch the two accounts for changes. We then get the balances of the two accounts and check if the sender has enough money. If they do, we start a new transaction and execute the two commands. If they do not, we unwatch the keys.

If another client modifies one of the watched keys before the `EXEC` command is executed, the transaction will be aborted.


---

### Quick Check

**You are in the middle of a transaction and you want to undo all the commands that you have queued so far. Which of the following would you use?**

   A. `EXEC`
-> B. **`DISCARD`**
   C. `UNWATCH`
   D. None of the above

<details>
<summary>See Answer</summary>

`DISCARD` is the correct choice for this task. It will undo all the commands that you have queued in the current transaction.

</details>

---

### Question 6: What is Lua scripting and how do you use it in Redis?

**Type:** Practical | **Category:** Lua scripting

## The Scenario

You are a backend engineer at a gaming company. You are building a new leaderboard service that needs to be able to update the score of a user and return their new rank in a single atomic operation.

You could do this by making two separate calls to Redis, but this would not be atomic.

## The Challenge

Explain what Lua scripting is in Redis and how you would use it to solve this problem. What are the key benefits of using Lua scripting?


> **Common Mistake:** A junior engineer might try to solve this problem by making two separate calls to Redis. This would not be atomic, and it could lead to a race condition.

> **Senior Engineer Approach:** A senior engineer would know that a Lua script is the perfect tool for this job. They would be able to explain what a Lua script is and how to use it to perform a complex, atomic operation.

### Step 1: Understand What Lua Scripting Is

Lua scripting is a feature of Redis that allows you to write complex, atomic operations in the Lua programming language.

### Step 2: The `EVAL` Command

The `EVAL` command is used to execute a Lua script.

```
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 mykey myvalue
```

### Step 3: Solve the Problem

Here's how we can use a Lua script to update the score of a user and return their new rank in a single atomic operation:

```lua
local score = redis.call('ZINCRBY', KEYS[1], ARGV[1], ARGV[2])
local rank = redis.call('ZREVRANK', KEYS[1], ARGV[2])
return {score, rank}
```

We can execute this script using the `EVAL` command:

```
EVAL "local score = redis.call('ZINCRBY', KEYS[1], ARGV[1], ARGV[2]); local rank = redis.call('ZREVRANK', KEYS[1], ARGV[2]); return {score, rank}" 1 leaderboard 100 user:123
```

### The Benefits of Using Lua Scripting

| Benefit           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Atomicity**     | A Lua script is executed as a single atomic operation.                                                  |
| **Performance**   | A Lua script is executed on the server, which can be faster than making multiple round trips from the client. |
| **Flexibility**   | Lua is a powerful scripting language that can be used to perform complex operations.                   |

---

### Question 7: What is Redis Sentinel and how do you use it for high availability?

**Type:** Practical | **Category:** High Availability

## The Scenario

You are a database administrator at a social media company. You are responsible for a Redis database that is critical to the business.

You have set up replication to a secondary database to provide high availability, but you need a way to automatically handle failover if the master node goes down.

## The Challenge

Explain what Redis Sentinel is and how you would use it to provide automatic failover for a Redis replica set. What are the key benefits of using Redis Sentinel?


> **Common Mistake:** A junior engineer might not be aware of Redis Sentinel. They might try to solve this problem by manually promoting a slave to be the new master, which would not be a very robust solution.

> **Senior Engineer Approach:** A senior engineer would know that Redis Sentinel is the perfect tool for this job. They would be able to explain what Redis Sentinel is and how to use it to provide automatic failover for a Redis replica set.

### Step 1: Understand What Redis Sentinel Is

Redis Sentinel is a system that provides high availability for Redis. It can automatically detect if the master node is down and promote one of the slave nodes to be the new master.

### Step 2: The Key Features of Redis Sentinel

| Feature      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **Monitoring** | Sentinel constantly monitors the health of the master and slave nodes.                                  |
| **Notification** | Sentinel can notify you if there is a problem with one of the nodes.                                    |
| **Automatic Failover** | Sentinel can automatically promote a slave to be the new master if the master goes down.         |
| **Configuration Provider** | Sentinel acts as a source of authority for clients service discovery.                           |

### Step 3: Set Up Redis Sentinel

Here's how we can set up a simple Redis Sentinel configuration with one master and two slaves:

**1. Configure the Sentinel servers:**

Create a `sentinel.conf` file on each Sentinel server.

```
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

**2. Start the Sentinel servers:**

Start the Sentinel servers.

```bash
redis-sentinel /path/to/sentinel.conf
```

**3. Connect your application to Sentinel:**

Your application should connect to the Sentinel servers instead of the Redis servers directly. The Sentinel servers will then provide your application with the address of the current master.


---

### Quick Check

**You have a three-node replica set and you want to be able to automatically promote a slave to be the new master if the master goes down. Which of the following would you use?**

   A. Replication
   B. Sharding
-> C. **Redis Sentinel**
   D. None of the above

<details>
<summary>See Answer</summary>

Redis Sentinel is the correct choice for this task. It is designed to provide automatic failover for a Redis replica set.

</details>

---

### Question 8: What is Redis Cluster and how do you use it for scalability?

**Type:** Practical | **Category:** Scalability

## The Scenario

You are a database administrator at a social media company. You are responsible for a Redis database that is growing very quickly. The database is starting to experience performance issues, and you have identified that the bottleneck is the single server that is hosting the database.

You need to find a way to scale the database horizontally to handle the increasing load.

## The Challenge

Explain what Redis Cluster is and how you would use it to provide horizontal scalability for a Redis database. What are the key benefits of using Redis Cluster?


> **Common Mistake:** A junior engineer might not be aware of Redis Cluster. They might try to solve this problem by just adding more resources to the single server, which would not be a very scalable solution.

> **Senior Engineer Approach:** A senior engineer would know that Redis Cluster is the perfect tool for this job. They would be able to explain what Redis Cluster is and how to use it to provide horizontal scalability for a Redis database.

### Step 1: Understand What Redis Cluster Is

Redis Cluster is a distributed implementation of Redis that allows you to automatically shard your data across multiple Redis nodes.

### Step 2: The Key Features of Redis Cluster

| Feature           | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| **Scalability**   | Redis Cluster allows you to scale your database horizontally by adding more nodes to the cluster.       |
| **High Availability** | Redis Cluster provides high availability by automatically failing over to a slave if a master node goes down. |
| **Performance**   | Redis Cluster can provide a significant performance improvement by distributing the load across multiple nodes. |

### Step 3: Set Up a Redis Cluster

Here's how we can set up a simple Redis Cluster with three masters and three slaves:

**1. Create the `redis.conf` files:**

Create a separate `redis.conf` file for each node in the cluster.

```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

**2. Start the servers:**

Start six Redis servers on different machines, each with its own `redis.conf` file.

**3. Create the cluster:**

Use the `redis-cli` tool to create the cluster.

```bash
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
--cluster-replicas 1
```

### Hash Slots

Redis Cluster uses a concept called hash slots to distribute the data across the shards. There are 16384 hash slots in a Redis Cluster.

When you set a key, Redis calculates a hash of the key and then uses the result to determine which hash slot the key belongs to. It then sends the key to the shard that is responsible for that hash slot.


---

### Quick Check

**You are designing a Redis Cluster for a new application. Which of the following would be the most important consideration when choosing a sharding strategy?**

   A. The number of shards.
   B. The number of replicas.
   C. The way that the data is distributed across the shards.
-> D. **All of the above**

<details>
<summary>See Answer</summary>

All of these are important considerations when choosing a sharding strategy. A good sharding strategy should have a sufficient number of shards and replicas, and it should distribute the data evenly across the shards.

</details>

---

### Question 9: What is the difference between `DEL` and `UNLINK` in Redis?

**Type:** Conceptual | **Category:** Core Concepts

## The Scenario

You are a backend engineer at a social media company. You are writing a new service that needs to delete a large number of keys from Redis.

You are not sure whether to use the `DEL` command or the `UNLINK` command.

## The Challenge

Explain the difference between the `DEL` and `UNLINK` commands in Redis. What are the pros and cons of each approach, and which one would you choose for this use case?


> **Common Mistake:** A junior engineer might think that they are interchangeable. They might not be aware of the performance implications of choosing one over the other.

> **Senior Engineer Approach:** A senior engineer would be able to provide a detailed explanation of the differences between `DEL` and `UNLINK`. They would also be able to explain the trade-offs between each approach and would have a clear recommendation for which one to use in this use case.

### Step 1: Understand the Key Differences

| Feature      | `DEL`                                                              | `UNLINK`                                                              |
| ------------ | ------------------------------------------------------------------ | --------------------------------------------------------------------- |
| **Blocking** | Blocks the server while the keys are being deleted.                | Does not block the server. The keys are deleted in the background.    |
| **Performance**| Can cause performance issues if you are deleting a large number of keys. | More performant than `DEL` for deleting a large number of keys.       |
| **Use Cases**  | When you need to delete a small number of keys.                    | When you need to delete a large number of keys.                       |

### Step 2: Choose the Right Tool for the Job

For our use case, we should use the **`UNLINK` command**. This is because we are deleting a large number of keys, and we do not want to block the server.

### Step 3: Code Examples

Here are some code examples that show the difference between the two approaches:

**`DEL`:**

```
DEL mykey1 mykey2 mykey3
```

**`UNLINK`:**

```
UNLINK mykey1 mykey2 mykey3
```


---

### Quick Check

**You need to delete a single key from Redis. Which of the following would be the most appropriate?**

   A. `DEL`
   B. `UNLINK`
-> C. **Either is fine**
   D. Neither is suitable for this task

<details>
<summary>See Answer</summary>

Either `DEL` or `UNLINK` would be fine for this task. The performance difference between the two is negligible for a single key.

</details>

---

### Question 10: What are Redis Streams and what are they used for?

**Type:** Conceptual | **Category:** Data Structures

## The Scenario

You are a backend engineer at a social media company. You are building a new real-time notification service.

You need to find a way to send notifications to users in a way that is reliable and scalable.

## The Challenge

Explain what Redis Streams are and how you would use them to solve this problem. What are the key benefits of using Redis Streams?


> **Common Mistake:** A junior engineer might try to solve this problem by using a Redis list. This would work, but it would not be a very robust solution. They might not be aware of Redis Streams, which is the correct tool for this job.

> **Senior Engineer Approach:** A senior engineer would know that Redis Streams are the perfect tool for this job. They would be able to explain what Redis Streams are and how to use them to build a reliable and scalable notification service.

### Step 1: Understand What Redis Streams Are

A Redis Stream is a data structure that is similar to a log file. It is an append-only data structure, which means that you can only add new entries to the end of the stream.

### Step 2: The Key Commands

| Command    | Description                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------- |
| **`XADD`**   | Adds a new entry to a stream.                                                                           |
| **`XREAD`**  | Reads one or more entries from a stream.                                                                |
| **`XGROUP`** | Creates, destroys, and manages consumer groups.                                                          |
| **`XREADGROUP`**| Reads from a stream via a consumer group.                                                               |

### Step 3: Solve the Problem

Here's how we can use Redis Streams to build a real-time notification service:

**1. Create a stream for each user:**

We can create a separate stream for each user to store their notifications.

**2. Add notifications to the stream:**

When a new notification comes in, we can use the `XADD` command to add it to the user's stream.

```
XADD notifications:123 * message "You have a new follower!"
```

**3. Read notifications from the stream:**

The user's client can then use the `XREAD` command to read the notifications from the stream.

```
XREAD COUNT 1 STREAMS notifications:123 0
```

### Consumer Groups

For a more robust solution, we can use consumer groups to allow multiple clients to read from the same stream. This is useful if a user is connected to the service from multiple devices.


---

### Quick Check

**You want to be able to read all the messages in a stream, starting from the beginning. Which of the following would you use?**

   A. `XADD`
-> B. **`XREAD`**
   C. `XGROUP`
   D. `XREADGROUP`

<details>
<summary>See Answer</summary>

`XREAD` is the correct choice for this task. You can use it to read one or more entries from a stream, starting from a specific ID.

</details>

---

### Question 11: What are bitmaps and hyperloglogs and what are they used for in Redis?

**Type:** Conceptual | **Category:** Data Structures

## The Scenario

You are a backend engineer at a social media company. You are building a new service that needs to track a large number of events, such as which users have liked a post or which users have seen an ad.

You need to find a way to store this data in a way that is efficient and that does not use a lot of memory.

## The Challenge

Explain what bitmaps and hyperloglogs are in Redis and how you would use them to solve this problem. What are the key benefits of using these data structures?


> **Common Mistake:** A junior engineer might try to solve this problem by using a set. This would work, but it would use a lot of memory for a large number of events. They might not be aware of bitmaps and hyperloglogs, which are much more memory-efficient.

> **Senior Engineer Approach:** A senior engineer would know that bitmaps and hyperloglogs are the perfect tools for this job. They would be able to explain what these data structures are and how to use them to track a large number of events in a memory-efficient way.

### Step 1: Understand What Bitmaps and HyperLogLogs Are

| Data Structure | Description                                                                                             | Use Cases                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Bitmaps**    | A bitmap is a data structure that allows you to store a boolean value for each element in a set.          | Tracking user activity, such as which users have liked a post or which users have seen an ad.           |
| **HyperLogLogs**| A HyperLogLog is a probabilistic data structure that is used to estimate the cardinality of a set.      | Counting the number of unique visitors to a website or the number of unique searches for a keyword. |

### Step 2: Choose the Right Tool for the Job

-   If you need to know whether a specific user has performed an action, you should use a **bitmap**.
-   If you only need to know the number of unique users who have performed an action, you should use a **HyperLogLog**.

### Step 3: Code Examples

Here are some code examples that show how to use these data structures:

**Bitmaps:**

```
# Mark user 123 as having seen the ad
SETBIT ad:1:seen 123 1

# Check if user 123 has seen the ad
GETBIT ad:1:seen 123

# Count the number of users who have seen the ad
BITCOUNT ad:1:seen
```

**HyperLogLogs:**

```
# Add a user to the set of unique visitors
PFADD unique_visitors "user:123"

# Get the number of unique visitors
PFCOUNT unique_visitors
```


---

### Quick Check

**You need to count the number of unique visitors to your website. Which of the following would be the most appropriate?**

   A. A set
   B. A bitmap
-> C. **A HyperLogLog**
   D. A sorted set

<details>
<summary>See Answer</summary>

A HyperLogLog is the correct choice for this task. It is a probabilistic data structure that is designed to estimate the cardinality of a set in a memory-efficient way.

</details>

---

### Question 12: How do you use Redis as a cache?

**Type:** Practical | **Category:** Caching

## The Scenario

You are a backend engineer at an e-commerce company. You are building a new service that needs to get product information from a database.

The database is slow, so you want to use Redis as a cache to speed up the service.

## The Challenge

Explain how you would use Redis as a cache. What are the key commands that you would use, and what are the different cache eviction policies that you can use?


> **Common Mistake:** A junior engineer might try to solve this problem by just storing the data in Redis without setting an expiration time. This would not be a very good solution, because the cache would eventually fill up and you would have to manually evict the old data.

> **Senior Engineer Approach:** A senior engineer would know that Redis is the perfect tool for this job. They would be able to explain how to use the `SET` command with the `EX` option to set an expiration time, and they would be able to explain the different cache eviction policies that are available.

### Step 1: Understand the Key Commands

| Command      | Description                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| **`SET`**    | Sets the value of a key.                                                                                |
| **`GET`**    | Gets the value of a key.                                                                                |
| **`EXPIRE`** | Sets an expiration time on a key.                                                                       |
| **`TTL`**    | Gets the remaining time to live of a key.                                                               |

### Step 2: Use Redis as a Cache

Here's how we can use Redis as a cache:

```python


r = redis.Redis()

def get_product(product_id):
    product = r.get(f"product:{product_id}")
    if product is None:
        product = get_product_from_db(product_id)
        r.set(f"product:{product_id}", product, ex=3600)
    return product
```

In this example, we first check if the product is in the cache. If it is, we return it. If it is not, we get it from the database, store it in the cache with an expiration time of 1 hour, and then return it.

### Step 3: Choose a Cache Eviction Policy

When the cache is full, Redis needs to evict some keys to make room for new ones. There are several different cache eviction policies that you can use:

| Policy                | Description                                                                                             |
| --------------------- | ------------------------------------------------------------------------------------------------------- |
| **`noeviction`**      | Don't evict any keys. Just return an error when the cache is full.                                      |
| **`allkeys-lru`**     | Evict the least recently used keys.                                                                     |
| **`allkeys-random`**  | Evict random keys.                                                                                      |
| **`volatile-lru`**    | Evict the least recently used keys that have an expiration time set.                                    |
| **`volatile-random`** | Evict random keys that have an expiration time set.                                                     |
| **`volatile-ttl`**    | Evict the keys that have the shortest time to live.                                                     |

You can configure the cache eviction policy in the `redis.conf` file.


---

### Quick Check

**You are using Redis as a cache and you want to evict the least recently used keys. Which of the following would you use?**

   A. `noeviction`
-> B. **`allkeys-lru`**
   C. `allkeys-random`
   D. `volatile-lru`

<details>
<summary>See Answer</summary>

`allkeys-lru` is the correct choice for this task. It will evict the least recently used keys, regardless of whether they have an expiration time set.

</details>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*

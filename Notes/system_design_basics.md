# Scalability Notes 

> this notes are just for my personal use and revision. it was created from the below repo and resorces specified in that repo. this is just my learnings and understandment from this repo which i wrote for clarity and revision 

# Part 1: Clones

- **Load Balancer**: Distributes requests across app servers. Requests from the same user may hit different servers.
- **Stateless Servers**: All servers must have the same codebase and store **no user data** locally (no sessions or files).
- **Session Management**: Use a centralized store like **Redis** or an **external DB**. Redis is preferred for speed.
- **Code Consistency**: Use deployment tools (e.g., **Capistrano**) to ensure all servers run the same version.
- **Server Cloning**: Create an AMI (Amazon Machine Image) from a configured server for easy, consistent scaling.


# Part 2: Database Bottlenecks

- **Initial Scalability Limitation**: App slows due to database (often MySQL) becoming a bottleneck.

## Path #1: Stick with MySQL
- Use **master-slave replication**: writes to master, reads from slaves.
- Keep scaling up master (more **RAM**).
- Long-term fixes include:
  - **Sharding**
  - **Denormalization**
  - **SQL tuning**
- Gets complex, costly, and harder over time.

## Path #2: Move to NoSQL Early
- **Denormalize early**: avoid joins entirely.
- Use MySQL like NoSQL, or switch to **MongoDB**, **CouchDB**, etc.
- Joins now done in **application code**.
- Needs initial effort but scales better.
- Eventually, you’ll need a **cache** to handle increasing read load.

> Tip: Migrate while the dataset is still small to save future refactoring time.


# Part 3: Caching

- **Problem**: Even with scalable DBs, user experience suffers due to slow reads for large data.
- **Solution**: Use **in-memory cache** (e.g., Redis or Memcached).
- Avoid **file-based caching** – makes auto-scaling and cloning hard.

## Cache Overview
- Acts as a **buffer layer** between app and DB.
- Store frequently-read data in cache to reduce DB load.
- RAM-based; extremely fast for both reads and writes (e.g., Redis can do 100K+ reads/sec).

## Caching Patterns

### 1. Cached Database Queries (Not Recommended)
- Cache the **result of a query**, using a **hashed query** as key.
- Problem: **Invalidation** is hard — any small DB update might require flushing multiple cache entries.

### 2. Cached Objects (Recommended)
- Cache **assembled data objects**, not raw queries.
- Example: Instead of caching queries for product data, cache the whole `Product` object after it's fully loaded.
- Easier to invalidate, easier logic, better performance.
- Enables **asynchronous processing**: worker servers can assemble objects → app just reads from cache.

## Good Candidates to Cache
- User sessions
- Fully rendered blog posts
- Activity feeds
- User-friend graphs

## Redis vs Memcached
- **Redis**: Richer features (persistence, data structures like lists, sets).
- **Memcached**: Lightweight, highly scalable, great for simple key-value caching.

> Caching is fast, simple to implement, and gives huge performance wins. Use it wisely!


# Part 4: Asynchronous Processing

- **Goal**: Avoid making users wait for slow tasks by using async workflows.
- Analogy: Don’t bake bread while the customer waits — prepare in advance or handle later.

## Async #1 – Precompute Ahead of Time
- "Bake at night, sell in morning" approach.
- Precompute heavy content (e.g., render dynamic pages into **static HTML**).
- Store/preload data periodically (e.g., via **cron jobs**).
- Host static content on **CDNs** (e.g., AWS S3/CloudFront) → huge scalability + speed.
- Best for general data that changes infrequently.

## Async #2 – Queue and Notify
- For on-demand, unpredictable tasks (e.g., "custom birthday cake").
- Flow:
  1. User initiates a heavy task.
  2. Frontend **queues** the task and gives immediate feedback ("Job started...").
  3. **Worker servers** listen to the queue and process jobs.
  4. Once done, notify the user (polling, WebSocket, email, etc.).

- Tools:
  - **RabbitMQ**, **ActiveMQ**, or **Redis lists** for job queues.
  - Worker services to process tasks in the background.

> Async = faster frontend, scalable backend, better UX.  
> Rule: Anything time-consuming → make it async!


## Keep in mind that everything is a trade-off.

# More about Scalability 

- **What is it that we really mean by scalability?**  A service is said to be scalable if when we increase the resources in a system, it results in increased performance in a manner proportional to resources added.

# Design Challenges in Distributed Systems

## Why Add Resources?
- To **scale performance**.
- To improve **reliability** via **redundancy** (e.g., fault tolerance, always-on services).

> A truly scalable system should **not degrade performance** when redundancy is introduced.

---

## Why Is Scalability Hard?

### 1. It Cannot Be an Afterthought
- Systems must be **designed for scalability** from the start.
- Many algorithms work fine on small scales but **break under:
  - High request rates
  - Large datasets
  - Increased number of distributed nodes

### 2. Dealing with Heterogeneity
- Scaling often brings **heterogeneous resources**:
  - Different hardware generations
  - Varying compute/storage capabilities
  - Geographically distributed nodes
- **Uniformity assumptions fail**:
  - Some nodes are faster/slower → inefficient usage
  - Algorithms assuming identical nodes often **underperform or break**

---

## Is Good Scalability Achievable?
  Yes, if you:
- Identify **growth axes** (e.g., users, data size, request rate)
- Design for **redundancy** without performance loss
- Build with **heterogeneity-awareness**
- Use the right tools/architectures and avoid common pitfalls

>  Scalability is not just about performance — it's about **growth without pain**.

# Different trade-offs

## Performance vs scalability
- A service is scalable if it results in increased performance in a manner proportional to resources added. Generally, increasing performance means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.1

Another way to look at performance vs scalability:

  - If you have a performance problem, your system is slow for a single user.
  - If you have a scalability problem, your system is fast for a single user but slow under heavy load.

---

## Latency vs Throughput

- **Latency**: Time taken to complete a single operation or request.
- **Throughput**: Number of operations completed per unit time (e.g., requests/sec).

> Goal: Achieve **higher throughput** with **lower latency**.

---
## Consistency, Availability, and Partition Tolerance (CAP Theorem)

In distributed systems, the **CAP theorem** states that it is impossible for a system to simultaneously guarantee:

-  **Consistency** – Every read receives the most recent write (all nodes see the same data).
-  **Availability** – Every request receives a response, even if some servers are down.
-  **Partition Tolerance** – The system continues to function despite network partitions (i.e., communication breakdowns between nodes).

>  A distributed system can provide at most **two out of the three** at any given time — not all three together.

---

### Simple Explanation

- **Consistency**: All servers reflect the same data at the same time — no matter which one you query.
- **Availability**: The system always responds to read/write requests — even if the data is slightly stale.
- **Partition Tolerance**: The system keeps running even if parts of the network are disconnected.

When a **partition** occurs (e.g., a server is unreachable), we have to make a trade-off:
-  If we prioritize **consistency**, we may have to **deny requests** until data is synced.
-  If we prioritize **availability**, we may allow **inconsistent reads/writes** across servers.

---

###  Real-World Analogy

If a server goes down:
- We might queue the work temporarily.
- But if it's unreliable or permanently offline, writing to it causes inconsistency.
- Therefore, systems must **choose between being consistent or available** during failures — not both.

---

###  Consistency Patterns

| Pattern             | Description                                                                 | Example Systems         |
|---------------------|-----------------------------------------------------------------------------|--------------------------|
| **Strong Consistency**   | All nodes return the same data instantly after a write.                      | Zookeeper, HBase         |
| **Eventual Consistency** | Data is eventually consistent across nodes, acceptable temporary lag.        | DynamoDB, Cassandra      |
| **Weak Consistency**     | No guarantees on read/write ordering or sync — high performance, low accuracy. | Caches, DNS              |

---

>  CAP theorem is a foundational concept in system design interviews and real-world distributed architecture. Choose your trade-offs wisely based on the system's goals.




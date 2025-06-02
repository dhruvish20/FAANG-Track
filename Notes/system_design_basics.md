# Scalability Notes 

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
- ❗Problem: **Invalidation** is hard — any small DB update might require flushing multiple cache entries.

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

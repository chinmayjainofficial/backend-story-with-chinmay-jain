# Caching: Detailed Notes

> Source: video transcript on what caching is, where it is used (Google, Netflix, Twitter/X), network/hardware/software caches, Redis, caching strategies, eviction policies, and backend use cases.

---

## Table of Contents

1. [What is Caching?](#1-what-is-caching)
2. [Example 1: Google Search](#2-example-1-google-search)
3. [Example 2: Netflix and CDN](#3-example-2-netflix-and-cdn)
4. [Example 3: Twitter/X Trending Topics](#4-example-3-twitterx-trending-topics)
5. [Three Levels of Caching](#5-three-levels-of-caching)
6. [Network-Level Cache: CDN](#6-network-level-cache-cdn)
7. [Network-Level Cache: DNS](#7-network-level-cache-dns)
8. [Hardware-Level Cache](#8-hardware-level-cache)
9. [Software-Level Cache: In-Memory Databases](#9-software-level-cache-in-memory-databases-redis-memcached)
10. [Caching Strategies](#10-caching-strategies)
11. [Eviction Policies](#11-eviction-policies)
12. [Practical Use Cases](#12-practical-use-cases-of-caching)
13. [Final Summary](#13-final-summary)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. What is Caching?

### One-line definition

> **Caching is a mechanism that reduces the time and effort needed to do some work.**

### Technical definition

Caching means keeping a **subset** of some data (not the whole thing) in a location that is **faster and easier to access** than the original (primary) storage.

### How do we choose which subset to cache?

It depends on:

- How **often** the data is used (frequency)
- The **probability** that it will be used again soon
- **Time** of use
- Many other parameters

### Why it matters

- This one technique is a **huge factor in high-performance applications**.
- Especially in systems that track latency in **milliseconds or even microseconds**.

### Key terms

| Term                | Meaning                                                                         |
| ------------------- | ------------------------------------------------------------------------------- |
| **Cache hit**       | The data you want **is found** in the cache (fast, happy path)                  |
| **Cache miss**      | The data is **not in the cache**, so you must fetch it from the original source |
| **Primary storage** | The original, full source of data (e.g., a database)                            |

---

## 2. Example 1: Google Search

### What happens when you search on Google?

- Your query goes through a complex workflow: **crawling, indexing, and ranking billions of web pages**.
- This is **computationally expensive** (lots of CPU, memory, and other resources).

### The problem without caching

- A query like **"what is the weather today"** is searched **millions of times a day**.
- Without caching, Google would **recompute everything for every single query**.
- Result: **very high latency** and **very high server load**. Each query would scan the index and run all ranking algorithms.

### How Google uses caching

- Uses a **distributed in-memory caching system**.
  - **Distributed** = cache servers are spread **across the whole world**, not in one place.
  - **In-memory** = data is stored in RAM (explained later).
- The **results** from all the ranking algorithms are **stored in the cache**.

### The workflow

```
User searches a query
        │
        ▼
Is the result in the cache?
        │
   ┌────┴────┐
  YES        NO
(cache hit)  (cache miss)
   │            │
   ▼            ▼
Return       Run the normal workflow
instantly    (ranking, sorting, etc.)
                │
                ▼
             Store result in cache
                │
                ▼
             Return to user
```

- On a **miss**, the result is computed and then **cached**, so the **next user** (same or different) searching the same query gets a **cache hit**.

---

## 3. Example 2: Netflix and CDN

### The challenge

Netflix delivers movies, series, and anime to **millions of users worldwide**, which means **terabytes** of data, with minimal buffering and without crashing servers.

### Encoding (quick background)

- Each video is **encoded into multiple resolutions** (e.g., 1080p, 720p, 480p).
- Netflix sends the **best version** for your **device** and **network speed**.
- This saves bandwidth and reduces load on Netflix servers.

### The solution: CDN (Content Delivery Network)

- Netflix has **origin servers** (e.g., in data centers in the US) that store all the actual movies.
- Netflix also places servers at many locations around the world, called **edge locations**.
  - They are "edge" because they are **strategically placed close to users** to **reduce latency**.

### Why this helps

- A user in the **US** is geographically close to US servers, so there is **low buffering**.
- A user in **India** would suffer high latency if everything came from the US origin server.
- With edge servers, the **nearest server** serves the content instead.

### CDNs are not just for Netflix

- Used by many small and big companies.
- Can serve **videos, images, web pages**, and **static assets** (JavaScript, HTML, CSS).
- Platforms like **Vercel** also have an **edge network**. Your request is served from the **closest region**.

### Connection to caching

- Netflix does **not** copy all its data to all edge locations. That would be too costly.
- It caches a **subset** (the content people in that region are likely to watch).
- It uses **trend analysis, ML algorithms, and real-time data** to decide what to cache.

### Why not cache everything?

- **Caching is more expensive** than disk storage.
- **Disk storage is cheaper** and has **more capacity**.
- Putting everything in the cache **defeats the purpose**.

---

## 4. Example 3: Twitter/X Trending Topics

_(Same idea applies to Facebook, LinkedIn, YouTube, etc.)_

### What X does

- Finds **trending topics** by analyzing **millions of tweets in real time**.
- Uses **ML algorithms, GPUs, and heavy infrastructure**, and processes **terabytes** of data.
- This calculation is **very expensive**.

### The problem without caching

If every user opening the trending section triggered this calculation, **servers would crash within seconds or minutes**.

### The solution

- Compute trending topics **every few minutes** (exact timing is unknown; this is just an estimate).
- Store the result in an **in-memory key-value store** (like **Redis**).
- When users request trends, serve them **directly from the cache**.

### Why it is safe to cache trends

- Trending topics don't change every second.
- Example: during elections, "elections" may trend for **hours or days**.

### The pattern (important!)

We use caching in **two common scenarios**:

1. When we want to **avoid doing heavy computation again and again**
2. When we want to **avoid sending large amounts of data** to many users repeatedly

---

## 5. Three Levels of Caching

Caching exists everywhere (hardware, software, browsers, and more). For backend engineers, the three most common are:

| #   | Level              | Examples                          |
| --- | ------------------ | --------------------------------- |
| 1   | **Network-level**  | CDN, DNS                          |
| 2   | **Hardware-level** | CPU caches (L1, L2, L3), RAM      |
| 3   | **Software-level** | Redis, Memcached, AWS ElastiCache |

> **Note:** "Software-based" caching isn't _purely_ software. It still uses hardware (RAM). It's called software-based because you interact with it through a **library or software**.

---

## 6. Network-Level Cache: CDN

### Core idea

Cache content on servers that are **geographically closer to users**. These are called:

- **Edge servers / Edge nodes / Edge computing**
- _(Anytime you see the word "edge", it means "the server closest to you".)_

### Benefits

- **Lower latency**
- **Less load / fewer resources** on the origin server

### How a CDN works (step by step)

1. **User requests a resource** (image, video, web page) by typing a URL in the browser.
2. **Browser sends a DNS query** to convert the domain name into an **IP address**.
3. **CDN's DNS system routes the request to the nearest PoP.**
   - **PoP (Point of Presence)** = a region with **multiple edge servers**. (Just a fancy term for a cluster of edge servers.)
   - Routing depends on:
     - User's **geographic location**
     - **Network conditions** (e.g., bad connection means it may route to a PoP with a lower-quality version of the video for less buffering)
   - Example: a user in **New York** is routed to a PoP near New York.
4. **Edge server checks its cache.**
   - **Cache hit** → serves the content immediately.
   - **Cache miss** → fetches it from the **origin server**, stores a copy, and serves it to the user.

### TTL (Time To Live)

- CDNs use **TTL** to decide **how long to keep content cached**.
- Example: cache for a few hours, since content may change or have regional versions.
- After TTL expires, the next request fetches **fresh content** from the origin, with a **new TTL**.

---

## 7. Network-Level Cache: DNS

### What is DNS?

DNS converts **domain names** (e.g., `example.com`) into **IP addresses**.

### Why caching is needed

Resolving a domain **recursively** through many servers is a lot of work. Caching means it only needs to be done **once for a period of time**.

### The full DNS flow (without cache)

```
You type example.com and press Enter
        │
        ▼
Device sends DNS query to a RECURSIVE RESOLVER
(provided by your ISP, or Google/Cloudflare public DNS)
        │
        ▼
Resolver checks its local cache ──► HIT? → return IP immediately
        │ MISS
        ▼
Ask a ROOT SERVER (about 13 sets, spread worldwide)
→ Root says: "I don't know the IP, but here's the .com TLD server"
        │
        ▼
Ask the TLD SERVER (.com)
→ Says: "Here's the authoritative name server for example.com"
        │
        ▼
Ask the AUTHORITATIVE NAME SERVER
→ Returns the actual IP address of example.com
        │
        ▼
Resolver returns IP to your device
```

### Key terms

| Term                          | Meaning                                                                                                              |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **ISP**                       | Internet Service Provider (e.g., Jio, ACT, Airtel)                                                                   |
| **Recursive resolver**        | Does the work of going from server to server until it finds the IP. Called "recursive" because it keeps going deeper |
| **Root servers**              | Don't know the IP, but know where the **top-level domain (TLD)** servers are (.com, .org, etc.)                      |
| **TLD server**                | Knows the address of the **authoritative name server** for a domain                                                  |
| **Authoritative name server** | Has the **actual IP** for the domain                                                                                 |

### Where caching happens in DNS (multiple layers!)

| Layer                             | Description                                                 |
| --------------------------------- | ----------------------------------------------------------- |
| 1. **Operating System**           | Windows/Mac/Linux keep a local DNS cache. Checked **first** |
| 2. **Browser**                    | Chrome, Firefox, etc. keep their **own** DNS cache          |
| 3. **Recursive resolver**         | ISP / Google / Cloudflare resolvers cache results           |
| 4. **Authoritative name servers** | Some (not all) also cache                                   |

If any layer has a **hit**, the rest of the chain is skipped.

---

## 8. Hardware-Level Cache

_(Familiar from computer science basics.)_

### The memory hierarchy

```
CPU
 │
L1 Cache   ← fastest, smallest
 │
L2 Cache
 │
L3 Cache   ← shared between CPU cores
 │
RAM (Main Memory)
 │
Secondary Storage (Hard Disk / SSD / Network storage)   ← slowest, biggest
```

### Why the CPU has caches

To store **repeated computations** and **frequently used data** so it can access them faster.

### Example: arrays

- Accessing array elements **sequentially** (1, 2, 3, 4, 5) is very fast.
- Reason: CPUs use **predictive algorithms**. They load the nearby block of memory into the L1/L2 cache ahead of time.

---

## 9. Software-Level Cache: In-Memory Databases (Redis, Memcached)

### Why RAM is fast

- **Hard disks** work **mechanically**. A head must move around the disk to find data.
- **RAM** uses **electrical signals** to access memory addresses **directly**.
- It's called **Random Access Memory** because access time is **almost constant**, no matter which address you access.

### RAM's trade-offs

| Property    | RAM (primary)                                 | Disk (secondary)            |
| ----------- | --------------------------------------------- | --------------------------- |
| Speed       | Very fast                                     | Slower                      |
| Capacity    | Limited                                       | Large                       |
| Cost        | More expensive                                | Cheaper                     |
| Persistence | **Volatile** (data is lost when power is off) | **Persistent** (data stays) |

> RAM trades **capacity and persistence for speed**, so it **cannot replace** disk storage.

### How Redis/Memcached handle persistence

- Data access (reading and modifying) happens in **RAM**, so it's fast.
- For persistence, they **also use secondary storage behind the scenes**.
- When the program starts, data is **loaded from disk into memory**.
- The caching technology itself is responsible for this persistence mechanism.

### Technologies

- **Redis**, **Memcached**
- Cloud versions: **AWS ElastiCache**
- Open-source alternative to Redis: **Valkey**

### Why they're called "in-memory key-value NoSQL databases"

| Part of the name | Meaning                                                                                                               |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| **In-memory**    | Data is stored in **RAM**, not on disk like PostgreSQL/MySQL                                                          |
| **Key-value**    | Simple structure: a **key** maps to a **value** (string, number, list, JSON, etc.). No tables, rows, or strict schema |
| **NoSQL**        | Doesn't enforce the strict structure of SQL databases                                                                 |

### Using them in code

- Just install a library for your language (e.g., `node-redis` for Node.js).
- **Store:** provide a key and a value.
- **Retrieve:** provide the key and get the value.
- No complex SQL queries or aggregations. It's simple.

---

## 10. Caching Strategies

### Strategy 1: Lazy Caching (Cache-Aside)

**Idea:** Only cache data **when someone actually requests it**.

```
Client requests a resource
        │
        ▼
Server checks the cache
   ┌────┴────┐
  HIT       MISS
   │          │
   │          ▼
   │     Fetch from primary storage (DB)
   │          │
   │          ▼
   │     Store it in cache
   │          │
   ▼          ▼
Return the result to the client
```

- Called "**lazy**" because you **don't predict or pre-load** data. You cache **on demand**.
- Very famous pattern in backend development.

### Strategy 2: Write-Through Caching

**Idea:** Whenever data changes (POST / PUT / PATCH), update the **database AND the cache at the same time** in the same API call.

| Pros                                            | Cons                                                            |
| ----------------------------------------------- | --------------------------------------------------------------- |
| Cache is **always fresh** (never stale/expired) | **Write operations have more overhead** (two updates each time) |

---

## 11. Eviction Policies

### Why we need them

- RAM is **limited**.
- Cache is only a **subset** of the data.
- When memory is full, we must **remove old data** to make room for new, more important data.
- An **eviction policy** decides **what to remove**.

### Types of eviction policies

#### 1. No Eviction

- No policy configured.
- When memory is full, new inserts give an **error** ("memory full").

#### 2. LRU: Least Recently Used

- Removes the key that was **accessed longest ago**.
- Example: Keys 1, 2, 3 were accessed today, and Key 4 was accessed yesterday. When Key 5 arrives and memory is full, **Key 4 is evicted**.

#### 3. LFU: Least Frequently Used

- Removes the key with the **lowest number of accesses**.
- Example:

| Key | Access count |
| --- | ------------ |
| 1   | 5            |
| 2   | 10           |
| 3   | 6            |
| 4   | 23           |

When Key 5 arrives, **Key 1 (5 accesses) is evicted**.

#### 4. TTL-Based Eviction

- Each key can have a **TTL (Time To Live)**.
- The key is automatically **invalidated** when its TTL runs out.
- During eviction, the key that will **expire soonest** is removed first.

### Quick comparison

| Policy          | Removes the key that...                          |
| --------------- | ------------------------------------------------ |
| **LRU**         | was **used least recently** (oldest access time) |
| **LFU**         | was **used least often** (lowest access count)   |
| **TTL**         | is going to **expire soonest**                   |
| **No eviction** | none. Throws an error when full                  |

---

## 12. Practical Use Cases of Caching

### 1. Database query caching

- **Scenario:** A heavy SQL query with many joins and aggregations, running on millions of rows, hit frequently (e.g., on a dashboard or landing page).
- **Problem:** High load on the database.
- **Solution:**
  1. Run the query **once**.
  2. Cache the result with a **TTL** (e.g., 1 hour).
  3. Serve later requests from the cache.
  4. If data changes, **update or delete** the cache entry.
- **Best when:** operations are **read-heavy** and writes are **infrequent**.

**Real-world examples:**

- **Amazon** caches **product details, prices, inventory**. A MacBook on sale may get millions of hits, but the product data rarely changes.
- **Twitter/Facebook** cache **user profile data**. Profiles change only a few times a year, but celebrity profiles might be fetched millions of times a day. On a change, they **invalidate** the cache and store the new entry.

### 2. Session storage

- After login, a **session token** is generated for the user.
- It must be checked on **every API request**.
- Storing it in **Redis** is much faster than the database, and **reduces DB load and latency**.

### 3. API caching (external APIs)

- **Scenario:** Your backend calls an external API (e.g., a weather API).
- **Problem without caching:** Thousands of calls leads to **higher billing** and hitting **rate limits**.
- **Solution:** Cache the response with a **TTL** (e.g., 1 hour). All requests in that hour use the cached data. After the TTL, the cache auto-invalidates and fresh data is fetched.
- Safe because weather data **doesn't change every second**.

### 4. Rate limiting

**How it works (step by step):**

1. A **rate-limit middleware** sits between the client and your route/controller.
2. It reads the client's **IP address** from the **`X-Forwarded-For`** header (usually added by a reverse proxy like Nginx).
3. It uses the IP as a **key** in Redis, with a **counter** as the value.
4. Every request **increments the counter**.
5. If the counter exceeds the limit (e.g., **50 requests per minute**), the request is **blocked** with **HTTP 429 (Too Many Requests)**.
6. The counter resets after the time window.

**Why use Redis and not a relational DB?**

- A DB call for **every request** adds **latency** (even 20 to 30 ms matters).
- It would **flood the database** (e.g., 1,000 users × 100 requests/min).
- Redis is **faster** and keeps the **load off the main DB**.

**Why rate limit?**

- Prevent **bot abuse**.
- Protect **compute-intensive APIs** and save resources.

---

## 13. Final Summary

1. **Caching** = keeping a **subset** of data in a **faster place** to save **time and effort**.
2. Use it when you want to avoid:
   - **Repeated heavy computation**, or
   - **Sending/fetching large data repeatedly**.
3. Three levels: **Network** (CDN, DNS), **Hardware** (L1/L2/L3, RAM), **Software** (Redis, Memcached).
4. **In-memory databases** are fast because they store data in **RAM**. They are **key-value** and **NoSQL**.
5. Two strategies: **Lazy caching (cache-aside)** and **Write-through**.
6. When the cache is full, an **eviction policy** is used: **No eviction, LRU, LFU, TTL**.
7. Common backend use cases: **DB query caching, session storage, API caching, rate limiting**.
8. **Next step:** Install **Redis** (or its open-source alternative **Valkey**), pick a library for your language, and try it out hands-on.

---

## 14. Quick Revision Cheat Sheet

| Concept                     | One-line summary                                                        |
| --------------------------- | ----------------------------------------------------------------------- |
| Caching                     | Store a subset of data in a faster place to save time and effort        |
| Cache hit                   | Data found in cache                                                     |
| Cache miss                  | Data not in cache; fetch from the source                                |
| CDN                         | Network of edge servers that cache content close to users               |
| Edge server / Edge location | Server geographically close to the user                                 |
| PoP                         | Point of Presence, a cluster of edge servers in a region                |
| Origin server               | Main server holding all the original content                            |
| TTL                         | Time To Live: how long cached data is valid                             |
| DNS                         | Converts domain names to IP addresses                                   |
| Recursive resolver          | Finds the IP by querying root, TLD, and authoritative servers           |
| DNS cache layers            | OS → Browser → Recursive resolver → Authoritative server                |
| L1/L2/L3                    | CPU hardware caches                                                     |
| RAM                         | Fast, limited, volatile memory used by in-memory caches                 |
| Redis / Memcached           | In-memory key-value NoSQL databases                                     |
| Valkey                      | Open-source alternative to Redis                                        |
| Lazy caching                | Cache on demand (cache-aside)                                           |
| Write-through               | Update DB and cache together; the cache stays fresh                     |
| LRU                         | Evict the least recently used key                                       |
| LFU                         | Evict the least frequently used key                                     |
| TTL eviction                | Evict the key that expires soonest                                      |
| Rate limiting               | Count requests per IP in Redis; block with 429 if the limit is exceeded |
| `X-Forwarded-For`           | Header carrying the client's public IP                                  |

### When should I cache?

```
Is the data read often but changed rarely?  ──► YES → good candidate
Is the computation heavy / expensive?       ──► YES → good candidate
Is the data huge and sent to many users?    ──► YES → good candidate
Does the data change every second?          ──► NO  → be careful / use a short TTL
```

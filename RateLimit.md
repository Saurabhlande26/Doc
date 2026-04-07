# 📄 Distributed Rate Limiter (Node.js + Redis + Load Balancer)

---

## 🧾 1. Overview

A **Distributed Rate Limiter** controls how many API requests a user can make within a given time window.

### Goals:

* Prevent API abuse
* Protect backend services
* Ensure fair usage
* Work across multiple servers

---

## 🎯 2. Problem Statement

In a multi-server setup:

```
User → Load Balancer → Server1 / Server2 / Server3
```

Each server maintains its own memory, causing:

* Inconsistent rate limiting ❌
* Users bypassing limits ❌

---

## ✅ 3. Solution

Use a **centralized Redis store** shared by all servers.

```
User → Load Balancer → App Servers → Redis (shared)
```

---

## 🧠 4. Algorithm

### Token Bucket Algorithm

* Each user has a bucket with tokens
* Each request consumes 1 token
* Tokens refill over time

---

## ⚙️ 5. Architecture

```
Client
 ↓
Load Balancer
 ↓
Node.js Servers (multiple)
 ↓
Redis (central store)
```

---

## 🗂️ 6. Components

### 1. Load Balancer

* Distributes traffic across servers

### 2. Application Servers

* Handle API requests
* Apply rate limiting

### 3. Redis

* Stores rate limit data
* Shared across all servers

---

## 🔑 7. Redis Data Model

### Key:

```
rate_limit:<userId>
```

### Value:

```json
{
  "tokens": 5,
  "lastRefill": 1710000000000
}
```

---

## 💻 8. Implementation (Node.js)

### Install Dependencies

```bash
npm install express redis
```

---

### Redis Setup

```js
const redis = require('redis');
const client = redis.createClient();

(async () => {
  await client.connect();
})();
```

---

### Rate Limiter Middleware

```js
const MAX_TOKENS = 10;
const REFILL_RATE = 1;

async function rateLimiter(req, res, next) {
  const userId = req.user?.id || req.ip;
  const key = `rate_limit:${userId}`;

  let data = await client.get(key);

  let bucket = data
    ? JSON.parse(data)
    : { tokens: MAX_TOKENS, lastRefill: Date.now() };

  const now = Date.now();
  const timePassed = (now - bucket.lastRefill) / 1000;

  const refill = Math.floor(timePassed * REFILL_RATE);
  bucket.tokens = Math.min(MAX_TOKENS, bucket.tokens + refill);
  bucket.lastRefill = now;

  if (bucket.tokens > 0) {
    bucket.tokens -= 1;

    await client.set(key, JSON.stringify(bucket), {
      EX: 60
    });

    next();
  } else {
    res.status(429).json({ message: "Too many requests" });
  }
}
```

---

## ⚠️ 9. Concurrency Issue

Multiple servers updating Redis simultaneously can cause:

* Duplicate token usage ❌
* Incorrect limits ❌

---

## 🔥 10. Solution: Atomic Operations (Lua Script)

Use Redis Lua script for atomic execution:

```js
const script = `
-- atomic token bucket logic here
`;
```

### Benefits:

* No race conditions
* Consistent updates
* Safe in distributed systems

---

## ⚡ 11. Enhancements

### 1. User-based Limits

```
Free User → 10 req/sec  
Premium → 100 req/sec  
```

---

### 2. TTL (Auto Cleanup)

```js
EX: 60
```

---

### 3. Logging

* Track blocked requests

---

### 4. Monitoring

* Redis metrics
* API usage trends

---

## 🔐 12. Best Practices

* Use JWT for user identification
* Keep Redis in same region as servers
* Use private network (VPC)
* Enable Redis replication

---

## ☁️ 13. Production Setup

* Load Balancer (ALB / Nginx)
* Auto Scaling servers
* Redis (managed service like ElastiCache)

---

## 🎯 14. Interview Summary

> I implemented a distributed rate limiter using the token bucket algorithm.
> Redis acts as a centralized store shared across multiple servers.
> Atomic operations ensure consistency and prevent race conditions.
> This design ensures scalable and reliable rate limiting.

---

## 🧠 15. Final Notes

### Advantages:

* Scalable
* Distributed
* Fast (in-memory Redis)
* Production-ready

### Use Cases:

* API protection
* Login throttling
* Payment endpoints
* Public APIs

---

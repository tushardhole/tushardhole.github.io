---
layout: post
title: "The 5-Minute System Design Survival Guide"
date: 2025-11-08 15:00:00 +0530
categories: [system-design, interview, tech]
tags: [observability, monitoring, scaling, databases, caching, message-queue, Kafka, RabbitMQ]
description: "A quick, no-nonsense guide to system design concepts you need before interviews. Observability, monitoring, scaling, databases, caching, and message queues explained in 5 minutes."
---

System design interviews can be intimidating, but a few **core concepts** will help you stay confident. Here’s a quick guide covering **observability, monitoring, scaling, databases, caching, and message queues**, with examples and diagrams.

---

## Observability vs Monitoring

**Observability** = Tools to understand how your system is performing and inspect logs, metrics, and traces.  
**Example tools:** Prometheus for metrics, Grafana for visualization, Kibana for logs.

**Monitoring** = Event-based alerts to catch issues proactively.  
**Example:** Alert when an API returns 500 or when the database is unavailable.

> Note: Grafana is a visualization tool; Prometheus actually **collects metrics** from your application.

---

## Quick Observability Diagram

```text
         +----------------+
         |      App       |
         | Exposes /metrics|
         +--------+-------+
                  |
                  v
         +----------------+
         |   Prometheus   |
         | Scrapes metrics|
         | every 15 secs  |
         +--------+-------+
                  |
                  v
         +----------------+
         |    Grafana     |
         | Visualizes     |
         | metrics        |
         +----------------+

         +----------------+
         |   Logstash     |
         | Processes logs |
         | and pushes to  |
         | Elasticsearch  |
         +--------+-------+
                  |
                  v
         +----------------+
         |    Kibana      |
         | Queries logs   |
         +----------------+
```
## Solving Scaling Issues

Scaling is about **handling more load efficiently**.

1. **Find the bottleneck:** Most of the time, it’s the database, but CPU, memory, or network can also be bottlenecks.
2. **Need more reads?** Use **read replicas** or **CQRS (Command Query Responsibility Segregation)**.
3. **Large data pages that change rarely:**
    - Example: Transactions page
    - Use **ETag headers** to return cached data if unchanged
4. **Global users:** Use a **CDN** to cache content closer to the user
5. **Write-heavy scenarios:** Consider **sharding, partitioning, or batching writes** to avoid DB contention

**Example workflow:**

```text
User requests transaction page
-> If ETag matches cache, return 304 Not Modified
-> Otherwise fetch from DB, cache, and return
```
## Message Queues

Message queues help **decouple services**, making your system more resilient and scalable. They allow one part of your system to send messages without needing the receiver to be immediately available.

| Use Case                  | Recommended Tool |
|----------------------------|----------------|
| Need replay/history        | Kafka          |
| Fire-and-forget messaging  | RabbitMQ       |

### Examples

**1. Kafka for event replay/history**

- Suppose you have two events: `UserSignedUp` and `UserPurchasedItem`.
- You want to generate analytics for users who made purchases shortly after signing up.
- Use **Kafka Streams** to join these events and process them asynchronously.

**2. RabbitMQ for fire-and-forget tasks**

- Example: Sending a welcome email after signup.
- The web server pushes the message to RabbitMQ, and a worker service consumes it to send the email.
- This prevents blocking the user request and handles retries transparently.

> **Tip for interviews:**
> - Kafka is optimized for **durable, replayable streams**, usually for analytics or event sourcing.
> - RabbitMQ is ideal for **task queues** where order and acknowledgement matter but replay is not needed.

---

## Choosing Databases

| Data Type                  | Recommended DB      | Reason                         |
|----------------------------|------------------|--------------------------------|
| Posts, comments            | NoSQL (MongoDB)   | Flexible schema, high write throughput |
| User profiles, transactions | SQL (Postgres/MySQL)| ACID compliance, structured data |

**Tips:**
- Use **NoSQL** for fast, scalable, unstructured data.
- Use **SQL** for multi-row transactions, joins, and strong consistency.
- Some NoSQL databases also support **single-document ACID transactions** if needed.

---

## Bonus: Caching Tips

- Cache **read-heavy** data to reduce DB load.
- Use **Redis** or **Memcached** for dynamic caching.
- Use **CDN (Content Delivery Network)** for static or cacheable content to reduce latency globally.
- **Cache invalidation:** Plan TTL (time-to-live) or versioned URLs for static assets to avoid serving stale data.

**When to use a CDN:**

1. **Static assets:** Images, CSS, JS files
    - Example: Blog logo or site stylesheets
2. **Large downloadable files:** PDFs, videos, or reports
    - Example: Whitepapers or system design cheat sheets
3. **Global user base:** Serve content closer to users geographically
    - Example: Users in Asia accessing a US-hosted blog
4. **Frequently accessed content:** Homepage banners, featured articles
    - Example: Popular posts that don’t change often
5. **API responses that rarely change:** Cache GET responses with proper headers
    - Example: `/latest-posts` API returning top 10 posts

**Example workflow with CDN + cache:**

```text
Request: /assets/logo.png
CDN checks cache:
-> If cached, return immediately to user
-> If not cached, fetch from origin server -> cache in CDN -> return response

Request: /latest-posts
Cache: Redis (dynamic) + CDN (optional)
-> If Redis cache hit, return cached data
-> If Redis cache miss, fetch from DB -> store in Redis -> serve to user
-> CDN caches the response for subsequent global requests
```

---
layout: post
title: "Part 2: Handling Orders, Backpressure, and Market Load in Real-Time Stock Price Systems"
description: "A continuation of our real-time stock price streaming system design. In this part, we dive into handling orders (limit/market), managing backpressure, and handling load during market open and close times."
date: 2025-11-19
justify: true
---

Part 2: Handling Orders, Backpressure, and Market Load in Real-Time Stock Price Systems

In **Part 1** of our blog series, we designed a scalable system for real-time stock price streaming, focusing on data ingestion, WebSockets/SSE, and data storage. In this second part, we dive into the challenges and strategies for handling **orders** (limit and market), **backpressure** during heavy trading periods, and managing **load during market open and close times**.

---

## 1. Types of Orders

In any trading system, **orders** represent user intent to buy or sell an asset (in this case, a stock). We commonly have two types of orders:

### 1.1 **Market Orders**

A **market order** is an order to buy or sell a stock immediately at the best available price. Market orders are typically filled instantly, but the price can fluctuate, especially when the stock is volatile or the order is large.

- **Characteristics:**
  - **Instant execution**
  - **Price uncertainty** (best available price)
  - **Low latency required**
  - **High volume** (for system load)

### 1.2 **Limit Orders**

A **limit order** specifies the maximum price a buyer is willing to pay (for a buy order) or the minimum price a seller is willing to accept (for a sell order). The order will not be executed until the market price reaches the specified limit price.

- **Characteristics:**
  - **Delayed execution** (only executed when market reaches limit)
  - **Price certainty** (you know the maximum/minimum price)
  - **Higher latency** (since it's waiting for a market match)
  - **Lower system load** (orders fill when the price hits the limit)

### 1.3 **How Orders are Processed**

When a market or limit order comes in, your backend needs to:
1. **Match market orders** with available liquidity (if they are market orders).
2. For **limit orders**, store them in an order book and **match them** when market prices hit the limit price.
3. Keep track of **open orders** in a queue or a database.

---

## 2. Backpressure Management in Real-Time Systems

### 2.1 **What is Backpressure?**

Backpressure happens when your system receives data faster than it can process it, leading to potential **overload**. In the context of trading systems, this might happen during periods of high trading volume, such as:
- The opening of the market
- High-impact news events (earnings, news, etc.)
- Large trades or institutional orders

If your system cannot process orders quickly enough, it can **drop orders**, cause **slowness**, or even experience **outages**.

### 2.2 **Handling Backpressure at Order Ingestion**

In real-time systems like stock exchanges or trading apps, managing **order flow** with **backpressure handling** is critical. Let’s look at strategies you can use:

#### 2.2.1 **Buffering and Queueing**

When your order ingestion system is overloaded, you can buffer incoming orders using a **queue** or a **message broker** like Kafka. This allows you to absorb high-frequency incoming orders and process them at a more controlled pace.

**Kafka** can be used to buffer incoming market and limit orders. This helps ensure that orders are not lost, even during peak traffic. The downstream processing system can consume messages from Kafka and handle the orders in a controlled way.

#### 2.2.2 **Rate Limiting and Throttling**

If you're receiving orders at a rate higher than you can process, you can **throttle** incoming requests. One way to do this is by applying a **rate limit** to the ingestion layer, such as:
- **Limit the number of requests per second** from each client
- **Prioritize market orders over limit orders**, as market orders require less processing time

This can help ensure that critical orders (market orders) are processed immediately, while non-urgent limit orders are queued for later processing.

#### 2.2.3 **Async Processing and Event-driven Architecture**

Handling order flow asynchronously helps maintain high throughput without blocking system resources. Using an **event-driven architecture** (such as **Akka Streams**, **Ktor’s coroutine-based async model**, or **Reactive Streams**) ensures that each order is processed concurrently without blocking the entire system.

---

## 3. Load Management During Market Open and Close

### 3.1 **Market Open (Heavy Load)**

The **market open** is the period when the stock market officially starts trading, typically marked by a spike in order volume. During this time, there’s often a flood of **market orders** to buy or sell assets.

### How to Handle Market Open Load:
- **Horizontal scaling**: Automatically scale the number of WebSocket/SSE server instances and Kafka consumers during peak trading hours.
- **Buffer orders**: Use Kafka to buffer incoming orders and avoid overloading the backend system.
- **Rate limiting**: Temporarily slow down order processing or prioritize specific types of orders (like market orders) during extremely busy periods.

### 3.2 **Market Close (High Volatility)**

The **market close** is another time of high trading volume, but the volume is typically accompanied by increased **volatility**. Traders might rush to close positions, and large institutional orders can spike up demand.

### How to Handle Market Close Load:
- **Queueing and event-driven processing**: Use Kafka and event-driven architectures to manage incoming orders during high volatility.
- **Batch processing**: Aggregate orders and execute them in batches to reduce the load on the matching engine.
- **Graceful degradation**: If load is too high, consider **pausing new order submissions** or prioritizing orders based on urgency.

---

## 4. Order Matching and Execution Engine

After receiving orders, we need to **match** market orders with existing limit orders in the order book. This involves:

- **Matching market orders** with limit orders at the best available price.
- **Executing the order** when a match is found, updating the order book, and notifying clients.
- **Handling order book state** (open limit orders, filled orders, etc.).

We can use a **Redis**-based order book for real-time updates, or a **TimescaleDB**/**ClickHouse** solution for persistent, historical order data.

---

## 5. System Design: High-Level Architecture

### 5.1 **System Components**
- **Order Ingestion Layer** (API / WebSockets / SSE)
  - Accepts limit and market orders from clients.
  - Implements rate limiting and basic validation.

- **Kafka**
  - Buffers incoming orders.
  - Ensures high availability and fault tolerance during periods of high traffic.

- **Order Matching Engine**
  - Matches market orders with available limit orders.
  - Executes trades and updates the order book.

- **Order Book**
  - Stores open limit orders (Redis for real-time, TimescaleDB/ClickHouse for persistence).

- **Execution Engine**
  - Handles the final execution of trades, notifying clients through WebSocket or SSE.

- **Backpressure Handling**
  - Uses rate limiting, queueing (Kafka), and async processing to manage load during high traffic periods.

### 5.2 **Diagram**

![](/images/stock-exchange-part-2.png)

# 6. Conclusion

In a high-frequency trading or stock market system, order handling, backpressure management, and market load management are all critical components to ensure smooth operation, especially during periods of high volume.

Here’s a quick recap:
- Limit orders are queued in the order book, waiting for a market match.
- Market orders are executed immediately at the best available price. 
- Backpressure handling using Kafka buffers, rate limiting, and async processing helps prevent overload. 
- Market open/close periods require special attention, such as horizontal scaling and buffering to prevent system failure during periods of extreme load.

By architecting a system that handles both the real-time ingestion of orders and the potential backpressure during high traffic, you can build a reliable and scalable trading platform.
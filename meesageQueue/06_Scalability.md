# Chapter 6: Scalability

## Introduction

যখন একটি System ছোট থাকে,

তখন একটি Producer, একটি Queue এবং একটি Consumer দিয়েই কাজ চলে।

কিন্তু যখন—

- User বেড়ে যায়
- Traffic বেড়ে যায়
- প্রতি সেকেন্ডে লাখ লাখ Message আসে

তখন একটি Queue বা একটি Consumer যথেষ্ট হয় না।

এই সমস্যা সমাধানের জন্য **Scalability** ব্যবহার করা হয়।

---

# What is Scalability?

Scalability হলো এমন একটি ক্ষমতা,

যার মাধ্যমে System বেশি Load Handle করতে পারে, Performance কমে না গিয়ে।

সহজ ভাষায়,

> **Traffic বাড়লেও System যেন ঠিকভাবে কাজ করতে পারে, সেটাই Scalability।**

---

## Example

ধরো,

একটি Email Service প্রতি মিনিটে 1000টি Email পাঠাতে পারে।

কিন্তু Black Friday-তে

100,000 Email পাঠাতে হবে।

তখন কী করবে?

আরও Consumer যোগ করবে।

আরও Queue Server যোগ করবে।

এটাই Scalability।

---

# Why Scalability is Important?

যদি System Scale না করতে পারে,

তাহলে—

- Response Slow হবে
- Queue বড় হতে থাকবে
- Consumer Overload হবে
- Message Delay হবে
- User Experience খারাপ হবে

---

# Before Scaling

```text
Producer

↓

Queue

↓

Consumer
```

একজন Consumer সব Message Process করছে।

---

# After Scaling

```text
             Producer
                 |
                 |
                 v
              Queue
                 |
      ---------------------
      |         |         |
      v         v         v
 Consumer1 Consumer2 Consumer3
```

এখন তিনজন Consumer একসাথে কাজ করছে।

---

# Types of Scaling

Scalability সাধারণত দুই ধরনের।

1. Vertical Scaling

2. Horizontal Scaling

---

# Vertical Scaling

Vertical Scaling মানে—

একই Server-কে আরও Powerful করা।

যেমন—

- বেশি CPU
- বেশি RAM
- বেশি Storage

```text
Before

Server

2 CPU

4GB RAM

↓

After

Server

16 CPU

64GB RAM
```

---

## Advantages

- Simple
- Configuration Change কম লাগে

---

## Disadvantages

- Expensive
- Hardware Limit আছে
- Single Point of Failure

---

# Horizontal Scaling

Horizontal Scaling মানে—

নতুন Server যোগ করা।

একটি Server বড় না করে,

অনেকগুলো Server ব্যবহার করা।

```text
Before

Queue Server
```

↓

```text
Queue Server 1

Queue Server 2

Queue Server 3
```

---

## Advantages

- Easy Scaling
- Better Performance
- High Availability
- Fault Tolerance

---

## Disadvantages

- একটু Complex
- Synchronization দরকার

---

# Distributed Queue

## What is Distributed Queue?

একটি Queue Server ব্যবহার না করে,

অনেকগুলো Queue Server ব্যবহার করা।

```text
              Producer
                  |
        --------------------
        |         |         |
        v         v         v
      Node1     Node2     Node3
```

সব Server মিলে Queue Handle করছে।

---

## Why Distributed Queue?

একটি Server Crash করলে,

অন্য Server কাজ চালিয়ে যাবে।

System বন্ধ হবে না।

---

## Examples

- Kafka Cluster
- RabbitMQ Cluster

---

# Partitioning

## What is Partitioning?

একটি বড় Queue বা Topic-কে ছোট ছোট ভাগে ভাগ করা।

এটাকে Partition বলে।

---

## Example

একটি Queue-তে

10 Million Message আছে।

তখন—

```text
Queue

↓

Partition 1

Partition 2

Partition 3
```

প্রতিটি Partition আলাদা Consumer Process করতে পারে।

---

## Diagram

```text
                Topic
                  |
     ----------------------------
     |            |             |
     v            v             v
 Partition1  Partition2  Partition3
```

---

## Kafka

Kafka এই Concept সবচেয়ে বেশি ব্যবহার করে।

একটি Topic-এর অনেকগুলো Partition থাকতে পারে।

---

## Advantages

- Parallel Processing

- Faster Processing

- Better Throughput

- Easy Scaling

---

# Consumer Group

Kafka Interview-এর জন্য খুব গুরুত্বপূর্ণ।

Consumer Group মানে—

একটি Group-এর মধ্যে অনেক Consumer থাকে।

একটি Partition একসাথে

একজন Consumer-ই Process করবে।

```text
Topic

↓

Partition1

↓

Consumer1

----------------

Partition2

↓

Consumer2

----------------

Partition3

↓

Consumer3
```

---

# Load Balancing

## What is Load Balancing?

সব Consumer-এর মধ্যে সমানভাবে Load ভাগ করে দেওয়া।

---

## Without Load Balancer

```text
Consumer1 = 10000 Messages

Consumer2 = 0

Consumer3 = 0
```

---

## With Load Balancer

```text
Consumer1 = 3300

Consumer2 = 3300

Consumer3 = 3400
```

সবাই সমানভাবে কাজ করছে।

---

## Diagram

```text
              Queue
                 |
      ----------------------
      |         |          |
      v         v          v
 Consumer1 Consumer2 Consumer3
```

---

# Queue Sharding

এটি Advanced Concept।

Sharding মানে—

Data-কে ভাগ করে

আলাদা Queue-তে রাখা।

উদাহরণ

```text
Users A-H

↓

Queue 1

----------------

Users I-P

↓

Queue 2

----------------

Users Q-Z

↓

Queue 3
```

---

## Advantage

- Less Load

- Better Performance

- Easy Scaling

---

# RabbitMQ Cluster

RabbitMQ-তে অনেকগুলো Node মিলে একটি Cluster তৈরি করে।

```text
           RabbitMQ Cluster

-------------------------------

Node1

Node2

Node3
```

যদি Node1 Down হয়,

Node2 কাজ চালিয়ে যাবে।

---

# Kafka Cluster

Kafka Broker-গুলো Cluster আকারে থাকে।

```text
Kafka Cluster

↓

Broker1

Broker2

Broker3
```

Topic-এর Partition বিভিন্ন Broker-এ থাকে।

---

# High Availability (HA)

High Availability মানে—

একটি Server Crash করলেও

System চালু থাকবে।

```text
Producer

↓

Node1 ❌

↓

Node2 ✅

↓

Consumer
```

User কোনো সমস্যা বুঝবে না।

---

# Scalability Techniques

| Technique          | Purpose             |
| ------------------ | ------------------- |
| Horizontal Scaling | নতুন Server যোগ করা |
| Vertical Scaling   | Server Upgrade করা  |
| Partitioning       | Queue ভাগ করা       |
| Load Balancing     | Load ভাগ করা        |
| Consumer Group     | Parallel Processing |
| Clustering         | High Availability   |
| Sharding           | Data ভাগ করা        |

---

# Real World Example

ধরো Amazon-এ

Black Friday Sale চলছে।

প্রতি সেকেন্ডে

১০ লক্ষ Order আসছে।

Amazon কী করবে?

- অনেক Queue ব্যবহার করবে।
- অনেক Consumer ব্যবহার করবে।
- Queue Partition করবে।
- Load Balancer ব্যবহার করবে।
- Cluster ব্যবহার করবে।

এভাবেই তারা কোটি কোটি Request Handle করে।

---

# Best Practices

✅ Horizontal Scaling Prefer করো।

✅ Queue ছোট রাখো।

✅ Multiple Consumer ব্যবহার করো।

✅ Load Balancer ব্যবহার করো।

✅ Cluster ব্যবহার করো।

✅ Partitioning ব্যবহার করো।

---

# Common Mistakes

❌ শুধু Vertical Scaling করা।

✔ Horizontal Scaling ব্যবহার করো।

---

❌ একটি Consumer ব্যবহার করা।

✔ Multiple Consumer ব্যবহার করো।

---

❌ সব Data এক Queue-তে রাখা।

✔ Partition বা Sharding ব্যবহার করো।

---

# Interview Questions

## What is Scalability?

Traffic বাড়লেও Performance ঠিক রেখে System-এর কাজ করার ক্ষমতা।

---

## Difference between Horizontal and Vertical Scaling?

Vertical → একই Server Upgrade করা।

Horizontal → নতুন Server যোগ করা।

---

## What is Partitioning?

একটি বড় Queue বা Topic-কে ছোট ছোট Partition-এ ভাগ করা।

---

## What is Load Balancing?

সব Consumer-এর মধ্যে Load সমানভাবে ভাগ করা।

---

## What is High Availability?

একটি Node Down হলেও System চালু থাকা।

---

## What is Queue Sharding?

Data-কে বিভিন্ন Queue-তে ভাগ করা।

---

## What is Consumer Group?

Kafka-তে একাধিক Consumer-এর Group, যারা Partition ভাগ করে Process করে।

---

# Summary

Scalability এমন একটি Technique, যার মাধ্যমে একটি Message Queue System বাড়তি Traffic ও Message সহজে Handle করতে পারে। এর জন্য Horizontal Scaling, Vertical Scaling, Distributed Queue, Partitioning, Consumer Group, Load Balancing, Sharding এবং Clustering ব্যবহার করা হয়। বড় বড় কোম্পানি যেমন Amazon, Netflix, Uber এবং Facebook এই Techniques ব্যবহার করে কোটি কোটি Request Process করে।

## Key Takeaways

- Scalability = More Traffic Handle করা।
- Horizontal Scaling = নতুন Server যোগ করা।
- Vertical Scaling = একই Server Upgrade করা।
- Partitioning = Queue/Topic ভাগ করা।
- Consumer Group = Parallel Processing।
- Load Balancer = কাজ ভাগ করে।
- Cluster = High Availability।
- Sharding = Data ভাগ করে Performance বাড়ানো।

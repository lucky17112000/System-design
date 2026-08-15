# Latency and Throughput

## 1. System Performance-এর দুটি গুরুত্বপূর্ণ Metric

System-এর performance মাপার জন্য দুটি গুরুত্বপূর্ণ metric হলো:

- **Latency**
- **Throughput**

সহজে মনে রাখো:

```text
Latency   → একটি কাজ কত দ্রুত শেষ হচ্ছে?
Throughput → নির্দিষ্ট সময়ে কতগুলো কাজ শেষ হচ্ছে?
```

---

## 2. Latency কী?

**Latency** হলো একটি operation সম্পন্ন হতে যে সময় লাগে।

> **একটি কাজ করতে কত সময় লাগে = Latency**

উদাহরণ:

```text
Click
  ↓
Request
  ↓
Server
  ↓
Response
  ↓
Complete

Latency = 200 ms
```

---

## 3. Network Latency কী?

**Network Latency** হলো network-এর মাধ্যমে data এক জায়গা থেকে অন্য জায়গায় যেতে যে সময় লাগে।

```text
Client
  │
  │ Request
  ▼
Server
  │
  │ Response
  ▼
Client
```

### Example

ধরো তুমি বাংলাদেশে এবং server Singapore-এ।

```text
Bangladesh Client
       │
       │ Request = 80 ms
       ▼
Singapore Server
       │
       │ Response = 80 ms
       ▼
Bangladesh Client
```

Round-trip delay:

```text
80 ms + 80 ms = 160 ms
```

> বাস্তবে total response time শুধু network latency-এর উপর নির্ভর করে না। Server processing, database query, queueing ইত্যাদিও সময় বাড়াতে পারে।

---

## 4. Latency কেন গুরুত্বপূর্ণ?

```text
Latency = 50 ms
```

হলে application দ্রুত মনে হবে।

কিন্তু:

```text
Latency = 5 seconds
```

হলে application slow মনে হবে।

---

## 5. সব System-এর জন্য Low Latency সবচেয়ে গুরুত্বপূর্ণ নয়

কিছু system-এ **Correctness, Reliability এবং Availability** বেশি গুরুত্বপূর্ণ।

### Example: Airline Booking System

```text
1 second response + Wrong booking = ❌
3 second response + Correct booking = ✅
```

একই seat দুইজনকে দেওয়া হলে দ্রুত response কোনো কাজে আসে না।

> কিছু system-এ **Correctness + Reliability > Extremely Low Latency**

---

## 6. Throughput কী?

**Throughput** হলো একটি system নির্দিষ্ট সময়ে কত amount of work process করতে পারে।

সহজভাবে:

> **নির্দিষ্ট সময়ে কতগুলো কাজ করা যায় = Throughput**

Web server-এর ক্ষেত্রে:

> প্রতি second-এ কতগুলো request process করা যায়।

Example:

```text
100 requests/second

Throughput = 100 req/sec
```

---

## 7. অনেক Request একসাথে এলে

ধরো:

```text
Server Capacity = 1000 req/sec
Incoming        = 5000 req/sec
```

সব request একসাথে process করা সম্ভব হবে না।

তখন queue তৈরি হতে পারে:

```text
Requests
   ↓
Queue
   ↓
Server
```

Queue বাড়লে waiting time বাড়তে পারে।

---

## 8. Throughput বাড়ানোর উপায়

### Vertical Scaling

একটি server-কে আরও powerful করা:

```text
4 CPU Core + 8 GB RAM
            ↓
16 CPU Core + 32 GB RAM
```

### Horizontal Scaling

আরও server যোগ করা:

```text
             Load Balancer
            /      |                 ↓       ↓       ↓
        Server 1 Server 2 Server 3
```

> বাস্তবে scaling সবসময় perfectly linear হয় না। Database, network, synchronization, load balancer ইত্যাদি bottleneck হতে পারে।

---

## 9. Latency বনাম Throughput

| Latency                        | Throughput                             |
| ------------------------------ | -------------------------------------- |
| একটি operation কত দ্রুত শেষ হয় | নির্দিষ্ট সময়ে কতগুলো operation শেষ হয় |
| Time-oriented                  | Capacity-oriented                      |
| ms/sec দিয়ে মাপা হয়            | req/sec, transactions/sec ইত্যাদি      |
| “একটি কাজ কত দ্রুত?”           | “এক সময়ে কত কাজ?”                      |

সহজে:

```text
Latency   = Speed
Throughput = Capacity
```

---

## 10. Restaurant Example

### Latency

একজন customer order দিল:

```text
Order → Food Ready
```

সময় লাগলো:

```text
10 minutes
```

এটা latency-এর মতো।

### Throughput

Restaurant এক ঘণ্টায়:

```text
200 orders
```

serve করতে পারে।

এটা throughput-এর মতো।

---

## 11. Latency এবং Throughput কি একই?

**না।**

```text
Low Latency ≠ High Throughput
High Throughput ≠ Low Latency
```

একটি system একটি request খুব দ্রুত শেষ করতে পারে, কিন্তু প্রতি second-এ খুব বেশি request process নাও করতে পারে।

আবার একটি system অনেক request process করতে পারে, কিন্তু প্রতিটি request-এর latency বেশি হতে পারে।

### Example

Server A:

```text
Latency    = 20 ms
Throughput = 100 req/sec
```

Server B:

```text
Latency    = 200 ms
Throughput = 10,000 req/sec
```

Server A দ্রুত individual response দিতে পারে।

Server B বেশি total work করতে পারে।

---

## 12. Load বাড়লে Latency কেন বাড়তে পারে?

Latency এবং Throughput আলাদা metric হলেও practical relationship থাকতে পারে।

```text
Many Requests
      ↓
   Queue
      ↓
   Server
```

Load বাড়লে:

```text
Load ↑
  ↓
Queue ↑
  ↓
Waiting Time ↑
  ↓
Latency ↑
```

তাই:

> **Latency এবং Throughput আলাদা হলেও high load-এর কারণে latency বাড়তে পারে।**

---

## 13. Performance Mental Model

```text
              SYSTEM PERFORMANCE
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
          Latency            Throughput
             │                   │
             ↓                   ↓
       How fast is          How much work
       one operation?       per unit time?
```

---

## 14. API Example

ধরো:

```text
Latency   = 100 ms
Throughput = 500 requests/sec
```

এর মানে:

- একটি request-এর response পেতে প্রায় 100 ms লাগতে পারে।
- system প্রতি second-এ প্রায় 500 request process করতে পারে।

---

## 15. Exam Definition

### Latency

> **Latency is the amount of time required for an operation to complete or for data to travel from one point to another.**

বাংলায়:

> **Latency হলো একটি operation সম্পন্ন হতে বা data এক point থেকে অন্য point-এ যেতে যে সময় লাগে।**

### Throughput

> **Throughput is the amount of work or number of operations a system can successfully process in a given period of time.**

বাংলায়:

> **Throughput হলো একটি system নির্দিষ্ট সময়ে কত পরিমাণ কাজ বা কতগুলো operation successfully process করতে পারে।**

---

## 16. Quick Revision

```text
Latency
=
একটি কাজ কত দ্রুত?

Network Latency
=
Network communication-এর delay

Throughput
=
একটি নির্দিষ্ট সময়ে কতগুলো কাজ?

Vertical Scaling
=
একটি server আরও powerful করা

Horizontal Scaling
=
আরও server যোগ করা
```

---

## 17. Final Memory Trick

```text
Latency
   ↓
TIME

Throughput
   ↓
CAPACITY
```

### সবচেয়ে গুরুত্বপূর্ণ লাইন

> **Latency measures time; Throughput measures capacity.**

আর:

> **Low latency থাকলেই high throughput হবে না, আর high throughput থাকলেই low latency হবে না। তবে system load বাড়লে queueing-এর কারণে latency বাড়তে পারে।**

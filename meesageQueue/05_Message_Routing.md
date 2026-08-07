# Chapter 5: Message Routing

## Introduction

একটি Message Queue System-এ Producer শুধু Message তৈরি করে।

কিন্তু Producer জানে না—

- Message কোন Queue-তে যাবে?
- কোন Consumer Message Process করবে?

এই সিদ্ধান্ত নেওয়ার Process-কে **Message Routing** বলা হয়।

সহজ ভাষায়,

> **Message Routing হলো এমন একটি Process, যা নির্ধারণ করে Producer-এর পাঠানো Message কোন Queue বা কোন Consumer-এর কাছে যাবে।**

```text
Producer
    |
    |
    v
Message Broker
    |
    |
Routing Decision
    |
    v
Queue
    |
    v
Consumer
```

---

# Why Message Routing?

ধরো একটি E-commerce Website আছে।

একজন Customer Order করল।

এখন একটি Order Event থেকে অনেকগুলো কাজ হবে।

- Email পাঠাতে হবে
- SMS পাঠাতে হবে
- Inventory Update করতে হবে
- Analytics Update করতে হবে

একটি Message থেকেই অনেক Service কাজ করবে।

```text
Order Service

↓

Message Broker

↓

Email Queue

SMS Queue

Inventory Queue

Analytics Queue
```

Routing-এর কাজ হলো—

সঠিক Queue-তে Message পাঠানো।

---

# Routing Flow

```text
Producer

↓

Message Broker

↓

Routing

↓

Queue

↓

Consumer
```

---

# Routing Methods

Message Routing-এর প্রধান ৩টি Method হলো—

1. Topic-Based Routing
2. Direct Routing
3. Content-Based Routing

---

# 1. Topic-Based Routing

## What is Topic-Based Routing?

Producer Message একটি **Topic**-এ Publish করে।

যে Consumer সেই Topic Subscribe করেছে,

সে Message পাবে।

---

## What is Topic?

Topic হলো একটি Category।

Example

```text
sports

payment

email

order

news

notification
```

---

## Example

Producer Publish করল—

```text
Topic = payment
```

এখন

```text
Payment Service

Analytics Service

Notification Service
```

এই তিনটি Service যদি payment Topic Subscribe করে,

তাহলে সবাই Message পাবে।

```text
              Producer
                  |
                  |
            payment Topic
          ________|_________
         |        |         |
         v        v         v
   Payment   Analytics   Notification
   Consumer   Consumer      Consumer
```

---

## Real World Example

YouTube

তুমি যদি Technology Channel Subscribe করো,

নতুন Video Upload হলে

Notification পাবে।

Subscribe না করলে পাবে না।

এটাই Topic-Based Routing।

---

## কোথায় ব্যবহার হয়?

- Apache Kafka
- Google Pub/Sub
- RabbitMQ Topic Exchange
- Redis Pub/Sub

---

## Advantages

- One-to-Many Communication
- Easy Scalability
- Event Driven Architecture

---

# 2. Direct Routing

## What is Direct Routing?

Producer আগে থেকেই জানে—

Message কোন Queue-তে যাবে।

Broker Routing Key দেখে

সরাসরি নির্দিষ্ট Queue-তে Message পাঠায়।

---

## Example

```text
Routing Key = email
```

↓

```text
Email Queue
```

---

আর যদি

```text
Routing Key = sms
```

↓

```text
SMS Queue
```

---

Diagram

```text
Producer
     |
Routing Key = email
     |
     v
Email Queue
     |
     v
Email Consumer
```

---

## Real World Example

ধরো Food Delivery App।

Order শুধুমাত্র যে Restaurant-এর,

সেই Restaurant-এই যাবে।

অন্য Restaurant পাবে না।

এটাই Direct Routing।

---

## কোথায় ব্যবহার হয়?

RabbitMQ Direct Exchange

---

## Advantages

- Fast
- Simple
- One Queue
- Less Complexity

---

# 3. Content-Based Routing

## What is Content-Based Routing?

এখানে Routing Key ব্যবহার করা হয় না।

Message-এর ভিতরের Data দেখে সিদ্ধান্ত নেওয়া হয়।

---

## Example

Producer পাঠালো—

```json
{
  "amount": 150000,
  "country": "Bangladesh"
}
```

Rule

```text
Amount > 100000
```

↓

VIP Queue

---

আর যদি

```text
Amount <=100000
```

↓

Normal Queue

---

আরেকটি Example

```json
{
  "priority": "HIGH"
}
```

↓

High Priority Queue

---

## Real World Example

Hospital

Patient এসেছে।

Heart Attack

↓

Emergency

---

Cold

↓

General Doctor

Patient-এর Condition দেখে Routing হচ্ছে।

---

## Advantages

- Smart Routing
- Flexible
- Business Rule Based

---

# RabbitMQ Exchange

এটি Interview-এর সবচেয়ে গুরুত্বপূর্ণ Concept।

Producer কখনো Queue-তে Direct Message পাঠায় না।

Producer প্রথমে Message Exchange-এ পাঠায়।

Exchange সিদ্ধান্ত নেয়—

Message কোন Queue-তে যাবে।

```text
Producer

↓

Exchange

↓

Queue

↓

Consumer
```

---

# Types of Exchange

RabbitMQ-তে ৪ ধরনের Exchange আছে।

- Direct Exchange
- Topic Exchange
- Fanout Exchange
- Headers Exchange

---

# 1. Direct Exchange

Routing Key Match হলে

Message নির্দিষ্ট Queue-তে যায়।

```text
Producer

↓

Direct Exchange

↓

Routing Key=email

↓

Email Queue
```

---

# 2. Topic Exchange

Pattern Match ব্যবহার করে।

Example

```text
order.*

payment.*

user.*
```

যদি

```text
order.created
```

Publish হয়,

তাহলে

```text
order.*
```

Subscriber Message পাবে।

---

# 3. Fanout Exchange

Routing Key Ignore করে।

সব Queue-তে Message পাঠায়।

```text
Producer

↓

Fanout Exchange

↓

Queue A

Queue B

Queue C
```

সব Queue একই Message পাবে।

---

## কোথায় ব্যবহার হয়?

Broadcast Notification

---

# 4. Headers Exchange

Routing Key ব্যবহার করে না।

Header দেখে সিদ্ধান্ত নেয়।

Example

```text
Priority = HIGH
```

↓

High Priority Queue

---

# Routing Comparison

| Routing Type  | Based On      | One/Many   |
| ------------- | ------------- | ---------- |
| Direct        | Routing Key   | One        |
| Topic         | Topic Pattern | Many       |
| Fanout        | Broadcast     | All        |
| Headers       | Header Value  | Rule Based |
| Content-Based | Message Body  | Rule Based |

---

# Real World Architecture

```text
                    Producer
                        |
                        |
                  RabbitMQ Exchange
                        |
      -----------------------------------
      |             |                  |
      v             v                  v
 Email Queue   SMS Queue     Notification Queue
      |             |                  |
      v             v                  v
 Email Worker   SMS Worker     Notification Worker
```

---

# Best Practices

✅ Routing Key Meaningful রাখো।

✅ ছোট ছোট Queue ব্যবহার করো।

✅ Fanout শুধুমাত্র Broadcast-এর জন্য ব্যবহার করো।

✅ Topic Exchange Event Driven System-এর জন্য ভালো।

---

# Common Mistakes

❌ Producer → Queue Directly

✔ Producer → Exchange → Queue

---

❌ সব Queue-এর জন্য একই Routing Key ব্যবহার করা।

✔ আলাদা Routing Key ব্যবহার করো।

---

❌ Fanout এবং Topic Exchange একই মনে করা।

✔ Fanout = Broadcast

✔ Topic = Pattern Matching

---

# Interview Questions

## What is Routing?

Message কোন Queue-তে যাবে সেই সিদ্ধান্ত নেওয়ার Process।

---

## What is Routing Key?

একটি Key যার মাধ্যমে Exchange নির্ধারণ করে Message কোন Queue-তে যাবে।

---

## Producer কি Queue-তে Direct Message পাঠায়?

RabbitMQ-তে সাধারণত না।

Producer প্রথমে Exchange-এ Message পাঠায়।

---

## What is Direct Exchange?

Routing Key Match করে Message নির্দিষ্ট Queue-তে পাঠায়।

---

## What is Topic Exchange?

Pattern Match ব্যবহার করে Message Route করে।

---

## What is Fanout Exchange?

সব Queue-তে একই Message Broadcast করে।

---

## What is Headers Exchange?

Header-এর Value দেখে Routing করে।

---

# Summary

Message Routing হলো Producer-এর পাঠানো Message কোন Queue বা কোন Consumer-এর কাছে যাবে তা নির্ধারণ করার Process।

RabbitMQ-তে Producer সাধারণত প্রথমে Exchange-এ Message পাঠায়। এরপর Exchange Routing Rule অনুযায়ী Message-কে এক বা একাধিক Queue-তে পাঠায়। Routing-এর জন্য Direct, Topic, Fanout এবং Headers Exchange ব্যবহার করা হয়।

## Key Takeaways

- Routing → Queue নির্বাচন করে।
- Exchange → Routing পরিচালনা করে।
- Direct → Routing Key।
- Topic → Pattern Matching।
- Fanout → Broadcast।
- Headers → Header Based।
- Content-Based → Message Body দেখে Routing।

# Chapter 3: Components of a Message Queue System

## Introduction

একটি Message Queue System একা কাজ করে না।

এখানে একাধিক Component একসাথে কাজ করে Message তৈরি করা, সংরক্ষণ করা এবং Process করার জন্য।

সবচেয়ে গুরুত্বপূর্ণ Components হলো—

- Producer
- Message Queue
- Consumer
- Message Broker (Optional কিন্তু খুব গুরুত্বপূর্ণ)

```text
               Producer
                   |
                   |
                   v
          +------------------+
          | Message Broker   |
          +------------------+
                   |
                   |
                   v
          +------------------+
          |  Message Queue   |
          +------------------+
                   |
          --------------------
          |        |         |
          v        v         v
      Consumer1 Consumer2 Consumer3
```

---

# 1. Producer

## What is Producer?

Producer হলো এমন একটি Service বা Application,

যা Message তৈরি করে Queue-তে পাঠায়।

Producer-এর কাজ শুধুমাত্র Message তৈরি করা।

Producer কখনো Message Process করে না।

---

## Real Life Example

ধরো Facebook-এ নতুন Account তৈরি করলে।

User Registration Service একটি Message তৈরি করবে—

```json
{
  "type": "SEND_EMAIL",
  "email": "rahim@gmail.com"
}
```

এই Message Queue-তে পাঠিয়ে Producer-এর কাজ শেষ।

---

## Examples of Producer

- User Service
- Payment Service
- Order Service
- Inventory Service
- Authentication Service
- Checkout Service

---

## Producer Workflow

```text
User

↓

Backend

↓

Producer

↓

Create Message

↓

Message Queue
```

---

## Important Points

✅ Producer Message তৈরি করে।

✅ Queue-তে Publish করে।

✅ Consumer-এর জন্য অপেক্ষা করে না।

✅ Asynchronous Communication নিশ্চিত করে।

---

# 2. Message Queue

## What is Message Queue?

Message Queue হলো এমন একটি Buffer,

যেখানে Producer-এর পাঠানো Message সাময়িকভাবে সংরক্ষণ করা হয়।

Consumer Ready হলে Queue Message Deliver করে।

---

## Responsibilities

Message Queue—

- Message Store করে
- Message Order Maintain করে
- Consumer Ready হলে Message দেয়
- Message Loss কমায়
- Buffer হিসেবে কাজ করে

---

## Example

ধরো—

১০,০০০ Email Request এসেছে।

Email Service প্রতি সেকেন্ডে ১০০টি Email পাঠাতে পারে।

Queue সব Message ধরে রাখবে।

```text
Producer

↓

Message Queue

↓

10000 Messages Waiting

↓

Consumer
```

---

## Popular Message Queue Systems

- RabbitMQ
- Apache Kafka
- AWS SQS
- Redis Streams
- ActiveMQ
- Azure Service Bus

---

# FIFO (First In First Out)

বেশিরভাগ Queue FIFO Follow করে।

যে Message আগে এসেছে,

সেটাই আগে Process হবে।

```text
Queue

↓

Message A

Message B

Message C
```

Consumer পাবে—

```text
Message A

↓

Message B

↓

Message C
```

---

# 3. Consumer

## What is Consumer?

Consumer হলো এমন একটি Service,

যা Queue থেকে Message নিয়ে Actual কাজ সম্পন্ন করে।

Consumer Queue Poll করে অথবা Queue থেকে Message Receive করে।

---

## Examples

- Email Service
- SMS Service
- Notification Service
- Analytics Service
- Image Processing Service

---

## Consumer Workflow

```text
Message Queue

↓

Consumer

↓

Process Message

↓

ACK
```

---

## Consumer Example

Queue-এর মধ্যে Message আছে—

```json
{
  "type": "SEND_EMAIL",
  "email": "rahim@gmail.com"
}
```

Consumer

↓

Email Service

↓

Nodemailer

↓

SMTP

↓

Email Sent

---

## Important Note

অনেকেই ভুল করে বলে—

"Nodemailer হচ্ছে Consumer"

এটি ভুল।

সঠিক Concept—

```text
Consumer

↓

Email Service

↓

Nodemailer

↓

SMTP Server
```

Consumer = Email Service

Nodemailer = Library

---

# Multiple Consumers

একটি Queue-এর জন্য একাধিক Consumer থাকতে পারে।

```text
              Queue
                 |
      --------------------
      |        |         |
      v        v         v
 Consumer1 Consumer2 Consumer3
```

এর সুবিধা—

- Faster Processing
- Parallel Processing
- Better Scalability

---

# 4. Message Broker

## What is Message Broker?

Message Broker হলো এমন একটি Software,

যা Producer এবং Consumer-এর মধ্যে Message আদান-প্রদান পরিচালনা করে।

Broker শুধু Message Store করে না,

বরং Message কোথায় যাবে সেটাও সিদ্ধান্ত নেয়।

---

## Responsibilities

Message Broker—

- Message Routing
- Queue Management
- Delivery
- Retry
- ACK Handling
- Dead Letter Queue
- Security
- Persistence

---

## Real World Example

Producer Order Completed Message পাঠালো।

```text
Order Completed
```

Broker সিদ্ধান্ত নিল—

```text
Order Completed

↓

Message Broker

↓

Email Queue

↓

SMS Queue

↓

Analytics Queue
```

---

## Examples of Message Broker

- RabbitMQ
- Apache Kafka
- ActiveMQ
- NATS
- Azure Service Bus

---

# Queue vs Broker

অনেকেই Queue এবং Broker-কে একই জিনিস মনে করে।

আসলে তারা এক নয়।

| Message Queue       | Message Broker     |
| ------------------- | ------------------ |
| Message Store করে   | Message Manage করে |
| Buffer              | Middleware         |
| অপেক্ষা করায়       | Routing করে        |
| Message Deliver করে | Queue Create করে   |
| FIFO Maintain করে   | Retry Handle করে   |

---

# RabbitMQ কি Queue নাকি Broker?

সবচেয়ে Common Interview Question।

Answer:

RabbitMQ একটি **Message Broker**।

RabbitMQ-এর ভিতরে অনেকগুলো Queue থাকতে পারে।

```text
RabbitMQ

├── Queue A

├── Queue B

├── Queue C
```

---

# Kafka কি?

Apache Kafka-ও একটি Message Broker।

তবে Kafka মূলত High Throughput Event Streaming Platform।

Kafka সম্পর্কে পরে আলাদা Chapter থাকবে।

---

# Complete Architecture

```text
                 Producer
                     |
                     |
                     v
            +------------------+
            | Message Broker   |
            +------------------+
                     |
         ----------------------------
         |            |             |
         v            v             v
     Queue A      Queue B      Queue C
         |            |             |
         v            v             v
     Consumer1    Consumer2    Consumer3
```

---

# Responsibilities of Each Component

| Component | Responsibility      |
| --------- | ------------------- |
| Producer  | Message তৈরি করে    |
| Broker    | Message Route করে   |
| Queue     | Message Store করে   |
| Consumer  | Message Process করে |

---

# Real World Example

ধরো একটি E-commerce Website।

User Order করল।

Order Service Message তৈরি করল।

↓

RabbitMQ

↓

RabbitMQ Message Route করল—

- Email Queue
- SMS Queue
- Analytics Queue

↓

প্রত্যেক Queue-এর Consumer আলাদা কাজ করল।

```text
Order Service

↓

RabbitMQ

↓

------------------------

↓

Email Queue

↓

Email Consumer

↓

Send Email

------------------------

↓

SMS Queue

↓

SMS Consumer

↓

Send SMS

------------------------

↓

Analytics Queue

↓

Analytics Consumer

↓

Update Analytics
```

---

# Best Practices

✅ Producer এবং Consumer আলাদা Service রাখো।

✅ Queue ছোট ছোট রাখো।

✅ Multiple Consumer ব্যবহার করো।

✅ ACK ব্যবহার করো।

✅ Retry Logic রাখো।

✅ DLQ ব্যবহার করো।

---

# Common Mistakes

❌ Producer দিয়ে Message Process করা।

✔ Producer শুধু Message Publish করবে।

---

❌ Consumer দিয়ে Message Publish করা।

✔ Consumer শুধু Message Process করবে।

---

❌ RabbitMQ-কে Queue ভাবা।

✔ RabbitMQ হলো Message Broker।

---

❌ Nodemailer-কে Consumer বলা।

✔ Email Service হলো Consumer।

---

# Interview Questions

## What is Producer?

Message তৈরি করে Queue-তে পাঠায়।

---

## What is Consumer?

Queue থেকে Message নিয়ে Process করে।

---

## What is Message Queue?

Producer-এর Message সাময়িকভাবে সংরক্ষণ করে।

---

## What is Message Broker?

Producer এবং Consumer-এর মধ্যে Message পরিচালনা করে।

Routing, Retry, ACK, Queue Management ইত্যাদি Handle করে।

---

## RabbitMQ কি Queue নাকি Broker?

RabbitMQ একটি Message Broker।

---

## Kafka কি?

Kafka একটি Distributed Message Broker এবং Event Streaming Platform।

---

# Summary

একটি Message Queue System-এর প্রধান Component হলো Producer, Message Queue, Consumer এবং Message Broker।

Producer Message তৈরি করে, Broker Message পরিচালনা ও Route করে, Queue Message সাময়িকভাবে সংরক্ষণ করে এবং Consumer সেই Message Process করে। এই Component-গুলো একসাথে কাজ করে একটি Scalable, Reliable এবং Asynchronous System তৈরি করে।

## Key Takeaways

- Producer → Message তৈরি করে।
- Broker → Message Route করে।
- Queue → Message Store করে।
- Consumer → Message Process করে।
- RabbitMQ → Message Broker।
- Multiple Consumer → Better Scalability।
- Producer এবং Consumer একে অপরের উপর Directly Depend করে না।

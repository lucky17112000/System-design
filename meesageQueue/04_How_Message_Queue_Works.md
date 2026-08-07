# Chapter 4: How Message Queue Works

## Introduction

এখন পর্যন্ত আমরা শিখেছি—

- Message Queue কী?
- Producer কী?
- Consumer কী?
- Broker কী?

এখন প্রশ্ন হলো—

> **Producer থেকে Consumer পর্যন্ত একটি Message কীভাবে পৌঁছায়?**

একটি Message সাধারণত নিচের ৪টি ধাপে Process হয়।

1. Sending Message
2. Queueing Message
3. Consuming Message
4. Acknowledgment (ACK)

```text
Producer
   |
   | (1. Send)
   v
+------------------+
|  Message Queue   |
+------------------+
   |
   | (2. Store)
   v
Consumer
   |
   | (3. Process)
   v
ACK
```

---

# Step 1: Sending Message

## What Happens?

Producer একটি Message তৈরি করে।

তারপর সেই Message Queue-তে Publish করে।

Producer-এর কাজ এখানেই শেষ।

Producer কখনো Consumer-এর জন্য অপেক্ষা করে না।

---

## Example

ধরো User Registration করেছে।

Producer Queue-তে নিচের Message পাঠালো।

```json
{
  "type": "SEND_EMAIL",
  "email": "rahim@gmail.com"
}
```

তারপর Producer সঙ্গে সঙ্গে User-কে Response দিয়ে দিল।

```text
Registration Successful
```

Email তখনও পাঠানো হয়নি।

---

## Why?

কারণ Email Background-এ যাবে।

এটাই Asynchronous Processing।

---

## Flow

```text
User

↓

Backend

↓

Producer

↓

Queue
```

---

# Step 2: Queueing Message

Producer Message পাঠানোর পরে,

Queue সেটি Store করে।

Queue Message ধরে রাখে যতক্ষণ না Consumer Ready হয়।

---

## Why Store?

ধরো—

১০,০০০টি Email Request এসেছে।

Consumer প্রতি সেকেন্ডে ১০০টি Email Process করতে পারে।

Queue সব Message ধরে রাখবে।

```text
Producer

↓

Queue

↓

Message 1

Message 2

Message 3

Message 4

Message 5

↓

Consumer
```

---

## FIFO

বেশিরভাগ Queue FIFO Follow করে।

FIFO = First In First Out

অর্থাৎ—

যে Message আগে এসেছে,

সেটাই আগে যাবে।

---

Example

```text
Queue

↓

Message A

Message B

Message C
```

Processing হবে—

```text
Message A

↓

Message B

↓

Message C
```

---

## Real Life Example

ব্যাংকের Token System

```text
Customer

↓

Token

↓

Waiting Area

↓

Cash Counter
```

Waiting Area-টাই Queue।

---

# Step 3: Consuming Message

Consumer Queue থেকে Message Receive করে।

তারপর Actual কাজ শুরু করে।

---

Example

Queue-এর মধ্যে Message আছে—

```json
{
  "email": "rahim@gmail.com",
  "subject": "Welcome"
}
```

Consumer—

```text
Queue

↓

Email Service

↓

Nodemailer

↓

SMTP

↓

Email Sent
```

---

Consumer নিজের Speed-এ কাজ করতে পারে।

Producer-এর সাথে তার কোনো Direct Connection নেই।

---

# Multiple Consumers

একটি Queue-এর জন্য অনেক Consumer থাকতে পারে।

```text
               Queue
                  |
     -------------------------
     |          |            |
     v          v            v
Consumer1 Consumer2 Consumer3
```

এর সুবিধা—

- Faster Processing
- Parallel Processing
- Scalability

---

# Step 4: ACK (Acknowledgment)

ACK হলো সবচেয়ে গুরুত্বপূর্ণ Concept।

Interview-এ প্রায়ই জিজ্ঞাসা করা হয়।

---

## What is ACK?

Consumer যখন সফলভাবে Message Process করে,

তখন Queue-কে জানায়—

> **"আমি Message Successfully Process করেছি।"**

এই Confirmation-কে বলা হয় ACK (Acknowledgment)।

---

## Flow

```text
Queue

↓

Consumer

↓

Process Message

↓

ACK

↓

Queue Deletes Message
```

---

# Why ACK is Important?

ধরো—

Consumer Message নিল।

কিন্তু Email পাঠানোর আগে Server Crash করল।

```text
Queue

↓

Consumer

↓

Crash
```

যদি ACK না থাকে—

Queue ভাববে Message Process হয়ে গেছে।

ফলে Message Delete হয়ে যাবে।

Email আর কখনো পাঠানো যাবে না।

---

ACK থাকলে—

Queue অপেক্ষা করবে।

```text
Queue

↓

Consumer

↓

Email Sent

↓

ACK

↓

Delete Message
```

এখন Message নিরাপদে Delete হবে।

---

# Retry Mechanism

যদি Consumer Message Process করতে না পারে,

Queue আবার সেই Message Process করার চেষ্টা করবে।

এটাকেই Retry বলে।

Example

```text
Try 1 ❌

Try 2 ❌

Try 3 ✅
```

তৃতীয়বার Success।

---

# Maximum Retry

ধরো Queue Rule দিয়েছে—

সর্বোচ্চ ৩ বার Retry করবে।

```text
Try 1 ❌

Try 2 ❌

Try 3 ❌
```

তিনবারই Fail।

↓

Message DLQ-তে যাবে।

---

# Requeue

কখনো Consumer Message Reject করতে পারে।

তখন Message আবার Queue-তে ফিরে আসে।

এটাকে Requeue বলে।

```text
Queue

↓

Consumer

↓

Reject

↓

Queue
```

পরে আবার অন্য Consumer Process করবে।

---

# Message Lifecycle

একটি Message-এর সম্পূর্ণ Journey।

```text
Producer

↓

Create Message

↓

Publish

↓

Queue

↓

Consumer

↓

Process

↓

ACK

↓

Delete
```

---

# Auto ACK vs Manual ACK

RabbitMQ-তে দুই ধরনের ACK আছে।

## Auto ACK

Consumer Message পেলেই Queue Message Delete করে দেয়।

```text
Queue

↓

Consumer

↓

Delete Immediately
```

### Problem

Consumer Crash করলে Message হারিয়ে যাবে।

---

## Manual ACK

Consumer কাজ শেষ করার পরে ACK পাঠায়।

```text
Queue

↓

Consumer

↓

Process

↓

ACK

↓

Delete
```

এটাই Production-এ ব্যবহার করা হয়।

---

# Which One Should You Use?

| Auto ACK              | Manual ACK |
| --------------------- | ---------- |
| Simple                | Safe       |
| Message Loss হতে পারে | Reliable   |
| Demo Project          | Production |

---

# Message Delivery Guarantees

Interview-এ অনেক জিজ্ঞাসা করা হয়।

---

## 1. At Most Once

Message একবারই Deliver হবে।

Lost হতে পারে।

Retry নেই।

---

## 2. At Least Once

Message অন্তত একবার Deliver হবে।

Retry আছে।

Duplicate হতে পারে।

RabbitMQ সাধারণত এই Model Follow করে।

---

## 3. Exactly Once

Message একবারই Deliver হবে।

Duplicate হবে না।

Lost হবে না।

এটি সবচেয়ে কঠিন Model।

Kafka কিছু Scenario-তে Exactly Once Support করে।

---

# Complete Workflow

```text
                Producer
                    |
                    |
             Create Message
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
          | Message Queue    |
          +------------------+
                    |
                    |
             Consumer Ready
                    |
                    |
                    v
             Process Message
                    |
          ------------------
          |                |
      Success           Failed
          |                |
          v                |
         ACK          Retry/Requeue
          |
          v
     Delete Message
```

---

# Real World Example

ধরো Amazon Order করেছ।

Order করার পরে—

Producer Queue-তে Message পাঠালো।

↓

Queue

↓

Order Consumer

↓

Inventory Update

↓

Payment Service

↓

Email Service

↓

SMS Service

সব Service Background-এ কাজ করছে।

তুমি Order করার সঙ্গে সঙ্গে Response পেয়ে গেছো।

---

# Best Practices

✅ Manual ACK ব্যবহার করো।

✅ Retry Logic রাখো।

✅ DLQ ব্যবহার করো।

✅ Long Running Task Queue-তে রাখো।

✅ Consumer Stateless রাখো।

---

# Common Mistakes

❌ Auto ACK Production-এ ব্যবহার করা।

✔ Manual ACK ব্যবহার করো।

---

❌ Retry Infinite রাখা।

✔ Maximum Retry Limit রাখো।

---

❌ ACK পাঠানোর আগে Message Delete করা।

✔ Process শেষ হলে ACK পাঠাও।

---

# Interview Questions

## How does Message Queue work?

Producer Message তৈরি করে → Queue Store করে → Consumer Process করে → ACK পাঠায় → Queue Message Delete করে।

---

## Why is ACK important?

ACK নিশ্চিত করে যে Message Successfully Process হয়েছে।

---

## What happens if Consumer crashes?

ACK না গেলে Queue Message Delete করবে না।

Retry অথবা Requeue করবে।

---

## What is Retry?

Failed Message আবার Process করার চেষ্টা।

---

## What is Requeue?

Rejected Message আবার Queue-তে পাঠানো।

---

## Auto ACK vs Manual ACK?

Production-এ সবসময় Manual ACK ব্যবহার করা উচিত।

---

# Summary

Message Queue-এর Working Process চারটি ধাপে সম্পন্ন হয়—

1. Producer Message পাঠায়।
2. Queue Message Store করে।
3. Consumer Message Process করে।
4. Consumer ACK পাঠালে Queue Message Delete করে।

যদি কোনো সমস্যা হয়, Queue Retry অথবা Requeue করতে পারে। Production System-এ সাধারণত Manual ACK ব্যবহার করা হয়, কারণ এটি Message Loss কমায় এবং System-কে আরও Reliable করে।

## Key Takeaways

- Producer → Send
- Queue → Store
- Consumer → Process
- ACK → Confirm
- Retry → Try Again
- Requeue → Send Back to Queue
- Manual ACK → Production Best Practice

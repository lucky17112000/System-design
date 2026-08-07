# Chapter 7: Dead Letter Queue (DLQ), Retry & Message Prioritization

## Introduction

বাস্তব জীবনে সব Message সফলভাবে Process হয় না।

কিছু Message—

- Invalid হতে পারে
- Consumer Crash করতে পারে
- Database Down হতে পারে
- Network Error হতে পারে
- Timeout হতে পারে

তাহলে সেই Message-এর কী হবে?

এই ধরনের Failed Message Handle করার জন্য

Message Queue System বিভিন্ন Technique ব্যবহার করে।

সবচেয়ে গুরুত্বপূর্ণ Concept হলো—

- Dead Letter Queue (DLQ)
- Retry
- Retry Queue
- Delay Queue
- Poison Message
- TTL (Time To Live)
- Message Prioritization

---

# What is Dead Letter Queue (DLQ)?

Dead Letter Queue (DLQ) হলো একটি Special Queue,

যেখানে সেইসব Message রাখা হয়

যেগুলো সফলভাবে Process করা যায়নি।

সহজ ভাষায়,

> **Main Queue-এর Failed Message-গুলো DLQ-তে পাঠানো হয়।**

---

## Why Do We Need DLQ?

ধরো Queue-তে 100টি Message আছে।

99টি ঠিক আছে।

কিন্তু একটি Message বারবার Fail করছে।

যদি Queue বারবার একই Message Process করতে থাকে,

তাহলে পিছনের সব Message আটকে যাবে।

এটাকে Blocking Problem বলে।

DLQ এই সমস্যা সমাধান করে।

---

## Without DLQ

```text
Queue

↓

Message 1 ✅

↓

Message 2 ❌

↓

Retry

↓

Retry

↓

Retry

↓

Retry

↓

Retry

↓

Everything Stops
```

---

## With DLQ

```text
Queue

↓

Message 1 ✅

↓

Message 2 ❌

↓

Maximum Retry Reached

↓

Dead Letter Queue

↓

Continue Processing

↓

Message 3 ✅

↓

Message 4 ✅
```

---

# When Does a Message Go to DLQ?

সাধারণত নিচের কারণগুলোতে Message DLQ-তে যায়।

---

## 1. Invalid Message Format

Example

Consumer আশা করছে—

```json
{
  "email": "rahim@gmail.com"
}
```

কিন্তু পেল—

```json
{
  "mail": "rahim@gmail.com"
}
```

Field Match করল না।

↓

Processing Failed

↓

DLQ

---

## 2. Maximum Retry Exceeded

ধরো Rule আছে—

Maximum Retry = 3

```text
Try 1 ❌

Try 2 ❌

Try 3 ❌
```

↓

DLQ

---

## 3. Message Expired (TTL)

Message অনেকক্ষণ Queue-তে পড়ে আছে।

TTL শেষ হয়ে গেছে।

↓

DLQ

---

## 4. Consumer Rejects the Message

Consumer নিজেই Message Reject করতে পারে।

↓

DLQ

---

## 5. Queue Does Not Exist

Routing ভুল হয়েছে।

Destination Queue নেই।

↓

DLQ

---

# DLQ Workflow

```text
Producer

↓

Main Queue

↓

Consumer

↓

Success

↓

ACK

------------------------

Failure

↓

Retry

↓

Retry

↓

Retry

↓

Dead Letter Queue
```

---

# Advantages of DLQ

- Main Queue Block হয় না।
- Failed Message হারিয়ে যায় না।
- পরে Debug করা যায়।
- চাইলে Reprocess করা যায়।
- Monitoring সহজ হয়।

---

# Retry Mechanism

সব Error Permanent নয়।

অনেক Error Temporary হয়।

যেমন—

- Database Down
- API Timeout
- Network Issue

এই ক্ষেত্রে Message আবার Process করা উচিত।

এটাকেই Retry বলে।

---

## Retry Example

```text
Try 1 ❌

↓

Wait

↓

Try 2 ❌

↓

Wait

↓

Try 3 ✅
```

---

# Retry Queue

Production System-এ অনেক সময় Main Queue-তে Retry করা হয় না।

বরং Retry Queue ব্যবহার করা হয়।

```text
Main Queue

↓

Consumer

↓

Failure

↓

Retry Queue

↓

Wait

↓

Main Queue
```

এর ফলে Main Queue Block হয় না।

---

# Exponential Backoff

সব Retry একসাথে করা উচিত নয়।

ধরো Database Down।

1 সেকেন্ড পর Retry

↓

2 সেকেন্ড পর Retry

↓

4 সেকেন্ড পর Retry

↓

8 সেকেন্ড পর Retry

↓

16 সেকেন্ড পর Retry

এটাকে Exponential Backoff বলে।

---

# Delay Queue

কিছু Message পরে Process করতে হয়।

Example

OTP

5 মিনিট পরে Expire হবে।

অথবা

Reminder Email

1 ঘণ্টা পরে যাবে।

এই ধরনের Message Delay Queue-তে রাখা হয়।

```text
Producer

↓

Delay Queue

↓

Wait

↓

Main Queue

↓

Consumer
```

---

# TTL (Time To Live)

TTL মানে—

একটি Message কতক্ষণ পর্যন্ত Valid থাকবে।

Example

TTL = 5 Minutes

5 মিনিট পরে Message Expire হবে।

---

## Example

OTP

TTL = 2 Minutes

2 মিনিট পরে OTP আর Valid না।

---

# Poison Message

Poison Message হলো—

যে Message কখনোই Successfully Process করা সম্ভব নয়।

Example

```json
{
  "age": "ABC"
}
```

Consumer Integer আশা করছে।

কিন্তু পেল String।

যতবার Retry করবে,

ততবারই Fail হবে।

↓

DLQ

---

# Message Prioritization

সব Message সমান গুরুত্বপূর্ণ নয়।

তাই Message-এর Priority Set করা যায়।

যার Priority বেশি,

সেটা আগে Process হবে।

---

## Example

Queue

```text
LOW

LOW

HIGH

MEDIUM
```

FIFO হলে—

LOW আগে যাবে।

কিন্তু Priority Queue হলে—

```text
HIGH

↓

MEDIUM

↓

LOW
```

---

# Priority Levels

সাধারণত—

```text
HIGH

MEDIUM

LOW
```

---

# Priority Based on Urgency

Hospital Example

```text
Heart Attack

↓

HIGH
```

---

```text
Fever

↓

LOW
```

---

# Priority Based on Business Rules

Amazon

VIP Customer

↓

HIGH

Regular Customer

↓

LOW

---

# Priority Based on Content

```json
{
  "priority": "HIGH"
}
```

↓

High Priority Queue

---

# FIFO vs Priority Queue

| FIFO                      | Priority Queue        |
| ------------------------- | --------------------- |
| First Come First Serve    | High Priority First   |
| Arrival Time গুরুত্বপূর্ণ | Priority গুরুত্বপূর্ণ |
| Simple                    | Flexible              |

---

# Real World Example

Food Delivery App

Orders

```text
VIP Customer

↓

Priority 10

----------------

Premium Customer

↓

Priority 7

----------------

Regular Customer

↓

Priority 2
```

Processing হবে—

VIP

↓

Premium

↓

Regular

---

# Complete Workflow

```text
                 Producer
                     |
                     |
                     v
               Main Queue
                     |
                     |
                Consumer
                     |
          ----------------------
          |                    |
          |                    |
      Success              Failure
          |                    |
          |                    |
         ACK               Retry Queue
                               |
                               |
                        Retry Limit Reached
                               |
                               |
                               v
                    Dead Letter Queue
```

---

# Best Practices

✅ Retry Limit রাখো।

✅ DLQ ব্যবহার করো।

✅ Poison Message Detect করো।

✅ Exponential Backoff ব্যবহার করো।

✅ Priority শুধুমাত্র Important Message-এর জন্য ব্যবহার করো।

---

# Common Mistakes

❌ Infinite Retry

✔ Maximum Retry রাখো।

---

❌ Failed Message Delete করা।

✔ DLQ-তে পাঠাও।

---

❌ সব Message HIGH Priority করা।

✔ Priority শুধুমাত্র Critical Message-এর জন্য।

---

# Interview Questions

## What is DLQ?

Failed Message Store করার Special Queue।

---

## Why is DLQ used?

Main Queue Block হওয়া বন্ধ করতে।

---

## What is Retry?

Failed Message আবার Process করার চেষ্টা।

---

## What is Retry Queue?

Retry করার আগে Message সাময়িকভাবে রাখা হয়।

---

## What is TTL?

Message কতক্ষণ Valid থাকবে।

---

## What is Poison Message?

যে Message কখনো Successfully Process করা যায় না।

---

## What is Delay Queue?

Future Time-এ Process করার Queue।

---

## What is Message Prioritization?

Priority অনুযায়ী Message আগে Process করা।

---

## FIFO vs Priority Queue?

FIFO → আগে আসলে আগে Process

Priority Queue → Priority বেশি হলে আগে Process

---

# Summary

Production System-এ সব Message সফলভাবে Process হয় না। তাই Failed Message Handle করার জন্য Dead Letter Queue (DLQ), Retry Mechanism, Retry Queue এবং Delay Queue ব্যবহার করা হয়। এছাড়া Message-এর গুরুত্ব অনুযায়ী Priority সেট করা যায়, যাতে গুরুত্বপূর্ণ Message আগে Process হয়।

## Key Takeaways

- DLQ = Failed Message Store করে।
- Retry = আবার Process করার চেষ্টা।
- Retry Queue = Retry-এর আগে Message ধরে রাখে।
- Delay Queue = পরে Process করার জন্য।
- TTL = Message-এর Lifetime।
- Poison Message = কখনো Process করা যায় না।
- Priority Queue = High Priority আগে Process হয়।
- FIFO = আগে আসলে আগে Process।

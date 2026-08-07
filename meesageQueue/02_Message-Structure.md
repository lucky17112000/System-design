# Chapter 2: Message Structure

## What is a Message?

Message Queue-তে **Producer** যে Data Queue-তে পাঠায়, সেটাকে **Message** বলা হয়।

একটি Message-এর মধ্যে সাধারণত দুই ধরনের তথ্য থাকে—

1. **Metadata (Message সম্পর্কে তথ্য)**
2. **Actual Data (যেটা Consumer Process করবে)**

তাই একটি Message সাধারণত **দুইটি অংশ** নিয়ে গঠিত।

- Header
- Body (Payload)

```text
Message
│
├── Header (Metadata)
│
└── Body (Payload)
```

---

# Message Structure

```text
Message
│
├── Header
│   ├── Message ID
│   ├── Timestamp
│   ├── Message Type
│   ├── Routing Information
│   ├── Priority
│   └── Correlation ID
│
└── Body
    ├── Actual Data
    ├── JSON
    ├── Text
    ├── Binary Data
    └── Files
```

---

# Header (Metadata)

## What is Header?

Header-এর মধ্যে Message সম্পর্কে Information থাকে।

এখানে Actual Data থাকে না।

Header-এর কাজ হলো Message-কে Identify করা এবং Queue-কে Message সম্পর্কে তথ্য দেওয়া।

সহজভাবে,

Header = Message-এর Identity Card

---

## Real Life Example

ধরো তুমি Courier-এ একটি Parcel পাঠাচ্ছ।

Parcel-এর বাইরে লেখা থাকে—

- Tracking Number
- Sender
- Receiver
- Delivery Address
- Delivery Priority

এই তথ্যগুলো হলো Header।

আর Box-এর ভিতরে যেটা আছে,

সেটা হলো Body।

```text
+--------------------------+
| Tracking ID              |
| Sender                   |
| Receiver                 |
| Priority                 |
+--------------------------+

Inside Box

Laptop
```

---

# What Information Does Header Store?

একটি Header-এ সাধারণত নিচের তথ্যগুলো থাকে।

---

## 1. Message ID

প্রত্যেক Message-এর একটি Unique ID থাকে।

উদাহরণ

```text
Message ID = MSG-1001
```

কেন দরকার?

- Duplicate Message Detect করতে
- Logging করতে
- Tracking করতে

---

## 2. Timestamp

Message কখন তৈরি হয়েছে।

```text
2026-08-08 10:35:21
```

কেন দরকার?

- Monitoring
- Logging
- Retry Logic
- TTL Check

---

## 3. Message Type

Message কী ধরনের?

Example

```text
SEND_EMAIL

SEND_SMS

CREATE_ORDER

PAYMENT_SUCCESS

USER_REGISTERED
```

Consumer এই Type দেখে বুঝতে পারে কী কাজ করতে হবে।

---

## 4. Routing Information

Message কোন Queue-তে যাবে।

Example

```text
Queue Name = EmailQueue
```

অথবা

```text
Routing Key = email
```

Broker এই তথ্য ব্যবহার করে Message Route করে।

---

## 5. Priority

Message কতটা গুরুত্বপূর্ণ।

Example

```text
Priority = HIGH
```

অথবা

```text
Priority = LOW
```

Priority Queue ব্যবহার করলে,

HIGH Priority Message আগে Process হবে।

---

## 6. Correlation ID

এটি Advanced Concept।

ধরো একটি Request-এর কারণে ৫টি Message তৈরি হয়েছে।

সবগুলোর Correlation ID একই হবে।

```text
Correlation ID

ABC-12345
```

এটি Distributed System-এ Request Track করতে ব্যবহৃত হয়।

---

# Example Header

```json
{
  "messageId": "MSG-1001",
  "timestamp": "2026-08-08T10:30:00",
  "type": "SEND_EMAIL",
  "priority": "HIGH",
  "routingKey": "email",
  "correlationId": "REQ-4567"
}
```

---

# Body (Payload)

## What is Body?

Body-এর মধ্যে থাকে Actual Data।

Consumer এই Data ব্যবহার করে কাজ করে।

Body-কে Payload-ও বলা হয়।

---

## Example

ধরো User Registration হয়েছে।

Producer Queue-তে নিচের Body পাঠালো।

```json
{
  "name": "Rahim",
  "email": "rahim@gmail.com",
  "subject": "Welcome",
  "message": "Thanks for joining."
}
```

Email Consumer এই তথ্য ব্যবহার করে Email পাঠাবে।

---

# Body কী কী Format হতে পারে?

Body অনেক ধরনের হতে পারে।

## JSON (Most Common)

```json
{
  "orderId": 101,
  "amount": 1200
}
```

---

## Plain Text

```text
Welcome User
```

---

## XML

```xml
<User>

<Name>Rahim</Name>

</User>
```

---

## Binary

- Image
- Audio
- Video

---

## File

- PDF
- Excel
- ZIP

---

# Complete Message Example

```text
Message
│
├── Header
│   ├── Message ID : MSG-1001
│   ├── Timestamp : 2026-08-08
│   ├── Type : SEND_EMAIL
│   ├── Priority : HIGH
│   ├── Routing Key : email
│   └── Correlation ID : REQ-456
│
└── Body
    ├── Email : rahim@gmail.com
    ├── Subject : Welcome
    └── Message : Thanks for joining
```

---

# Real World Example

ধরো একটি Food Delivery App।

Customer খাবারের Order দিল।

Queue-তে একটি Message যাবে।

## Header

```text
Order ID = 1001

Priority = HIGH

Timestamp = 11:20 AM

Restaurant = KFC
```

## Body

```json
{
  "customer": "Rahim",
  "items": ["Burger", "Coke"],
  "address": "Dhaka"
}
```

Restaurant Body পড়ে খাবার তৈরি করবে।

---

# Why Separate Header and Body?

অনেকেই প্রশ্ন করে—

সব Data একসাথে রাখলেই তো হয়।

তাহলে Header আলাদা কেন?

কারণ—

Header Broker এবং Queue ব্যবহার করে।

Body Consumer ব্যবহার করে।

Broker-এর দরকার নেই Email Content জানার।

Broker শুধু জানে—

```text
Routing Key

Priority

Queue Name
```

এগুলো Header-এ থাকে।

---

# Message Flow

```text
Producer

↓

Create Message

↓

Header + Body

↓

Message Queue

↓

Consumer

↓

Read Header

↓

Read Body

↓

Process
```

---

# Header vs Body

| Header                   | Body                            |
| ------------------------ | ------------------------------- |
| Metadata                 | Actual Data                     |
| Queue ব্যবহার করে        | Consumer ব্যবহার করে            |
| Routing Information থাকে | Business Data থাকে              |
| Priority থাকে            | User Data থাকে                  |
| Message Type থাকে        | Email, Order, Payment Data থাকে |

---

# Best Practices

✅ Header ছোট রাখো।

✅ Body-তে Actual Business Data রাখো।

✅ JSON Format ব্যবহার করো।

✅ প্রতিটি Message-এর Unique ID রাখো।

✅ Timestamp রাখো।

✅ Correlation ID ব্যবহার করো (Microservices-এর জন্য)।

---

# Common Mistakes

❌ Header-এর মধ্যে পুরো Business Data রাখা।

✔ শুধু Metadata রাখো।

---

❌ Message Type না রাখা।

✔ সবসময় Type রাখো।

---

❌ Message ID না রাখা।

✔ Tracking-এর জন্য Unique ID রাখো।

---

# Interview Questions

## What is a Message?

Producer Queue-তে যে Data পাঠায়, সেটাই Message।

---

## What are the two main parts of a Message?

- Header
- Body (Payload)

---

## What is Header?

Header Message-এর Metadata Store করে।

---

## What is Body?

Body Actual Business Data Store করে।

---

## What is Metadata?

Message সম্পর্কে Information।

যেমন—

- Message ID
- Timestamp
- Routing Key
- Priority

---

## What is Payload?

Payload হলো Message-এর Actual Data।

---

## Why is Message divided into Header and Body?

কারণ—

Header Queue/Broker ব্যবহার করে।

Body Consumer ব্যবহার করে।

এতে System আরও Efficient এবং Flexible হয়।

---

# Summary

একটি Message সাধারণত **Header** এবং **Body**—এই দুইটি অংশ নিয়ে গঠিত।

Header-এ Message সম্পর্কে Metadata থাকে, যেমন Message ID, Timestamp, Routing Key, Priority ইত্যাদি। Broker ও Queue এই তথ্য ব্যবহার করে Message পরিচালনা করে।

Body-তে Actual Business Data থাকে, যেমন Email, Order, Payment বা User Information। Consumer এই Data পড়ে প্রয়োজনীয় কাজ সম্পন্ন করে।

## Key Takeaways

- Message = Header + Body
- Header = Metadata
- Body = Payload
- Header Broker ব্যবহার করে
- Body Consumer ব্যবহার করে
- JSON হলো সবচেয়ে জনপ্রিয় Payload Format
- প্রতিটি Message-এর Unique ID থাকা উচিত
- Timestamp এবং Correlation ID Distributed System-এ খুব গুরুত্বপূর্ণ

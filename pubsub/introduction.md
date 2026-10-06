# 📡 Publish-Subscribe (Pub/Sub) — System Design Notes

> **Pub/Sub (Publish-Subscribe)** হলো একটি messaging pattern যেখানে **Publisher** সরাসরি **Subscriber**-কে message পাঠায় না। Publisher একটি **Topic**-এ message publish করে, আর Subscriber সেই Topic থেকে message consume করে।

### 🔑 Core Idea

```text
Publisher → Topic → Subscriber
```

---

# 🧩 1. Pub/Sub-এর 4টি Key Entity

```text
1. Publisher
2. Subscriber
3. Topic
4. Message
```

```mermaid
flowchart LR
    P[📤 Publisher] -->|📩 Publish Message| T[(🟪 Topic)]
    T -->|Deliver| S1[📥 Subscriber 1]
    T -->|Deliver| S2[📥 Subscriber 2]
    T -->|Deliver| S3[📥 Subscriber 3]

    classDef pub fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef topic fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef sub fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;

    class P pub;
    class T topic;
    class S1,S2,S3 sub;
```

## Publisher

**Publisher** হলো যে service/component message বা event publish করে।

Example:

```text
Order Service
Payment Service
Chat Service
Notification Service
```

## Subscriber

**Subscriber** হলো যে service কোনো Topic-এর message receive বা consume করে।

Example:

```text
Email Service
Inventory Service
Analytics Service
Notification Service
```

## Topic

**Topic হলো Publisher এবং Subscriber-এর মাঝের intermediary/channel।**

## Message

**Message হলো actual data/event যা Publisher Topic-এ publish করে।**

Example:

```json
{
  "orderId": 123,
  "userId": 45,
  "status": "created"
}
```

---

# 🔵 2. Basic Streaming বনাম Pub/Sub

Basic streaming:

```text
Client ←────────→ Server
       Direct Connection
```

Pub/Sub:

```text
Publisher
    ↓
  Topic
    ↓
Subscriber
```

```mermaid
flowchart LR
    subgraph A["🔵 Basic Streaming"]
        C[👤 Client] <-->|🔗 Direct Connection| S[🖥️ Server]
    end

    subgraph B["🟢 Pub/Sub"]
        P[📤 Publisher] --> T[(🟪 Topic)] --> U[📥 Subscriber]
    end

    classDef blue fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef green fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;
    classDef purple fill:#ede9fe,stroke:#7c3aed,stroke-width:3px,color:#111827;

    class C,S blue;
    class P,U green;
    class T purple;
```

> **Main Difference:** Pub/Sub-এ Topic একটি intermediary হিসেবে কাজ করে।

---

# 🔗 3. Publisher সরাসরি Subscriber-কে পাঠায় না কেন?

ধরো একটি e-commerce system-এ order তৈরি হলো।

একই event-এ interested:

```text
Email Service
Inventory Service
Analytics Service
Notification Service
```

Direct communication হলে:

```text
Order Service
 ├──→ Email Service
 ├──→ Inventory Service
 ├──→ Analytics Service
 └──→ Notification Service
```

এতে Publisher-কে অনেক consumer-এর সাথে direct dependency maintain করতে হয়।

এটা **tight coupling** তৈরি করতে পারে।

---

# 🟢 4. Pub/Sub কীভাবে Decoupling করে?

Pub/Sub-এ:

```mermaid
flowchart TD
    O[🛒 Order Service] -->|order-created| T[(🟪 order-created Topic)]
    T --> E[📧 Email]
    T --> I[📦 Inventory]
    T --> A[📊 Analytics]
    T --> N[🔔 Notification]

    classDef order fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef topic fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef svc fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;

    class O order;
    class T topic;
    class E,I,A,N svc;
```

Order Service শুধু জানে:

> **“আমি `order-created` topic-এ publish করবো।”**

Publisher-এর subscriber-এর identity জানার প্রয়োজন নেই।

এটাই:

> **Loose Coupling / Decoupling**

---

# 🌐 5. Distributed Systems-এ কেন গুরুত্বপূর্ণ?

Large-scale distributed system-এ অনেক service একসাথে কাজ করে।

```text
Service A
Service B
Service C
Service D
Service E
```

সব service direct communication করলে complexity দ্রুত বাড়ে।

Pub/Sub:

```text
Services
   ↓
 Topics
   ↓
Services
```

ফলে system আরও modular হতে পারে।

```text
✅ Loose Coupling
✅ Scalability
✅ Flexibility
✅ Easier Integration
```

---

# 🌩️ 6. Network Failure / Server Failure

Distributed system-এ network বা consumer server temporarily fail করতে পারে।

Direct connection-এর ক্ষেত্রে message delivery fail হতে পারে।

Pub/Sub-এ Topic যদি message retain/persist করতে পারে:

```text
Publisher
    ↓
  Topic
    ↓
📦 Message Stored
    ↓
Subscriber ❌
```

Subscriber পরে ফিরে এসে retained message consume করতে পারে।

```mermaid
sequenceDiagram
    participant P as 📤 Publisher
    participant T as 🟪 Topic
    participant C as 📥 Consumer

    P->>T: Publish Event
    Note over T: Message retained
    T--xC: Consumer unavailable

    C->>T: Reconnect
    T->>C: Deliver retained Event
    C->>T: ACK ✅
```

> Exact retention/durability behavior messaging system-এর implementation ও configuration-এর উপর নির্ভর করে।

---

# 📦 7. Message Persistence

Message topic/message store-এ retained থাকতে পারে।

Example:

```text
Topic
 ├── Message 101
 ├── Message 102
 └── Message 103
```

Subscriber temporarily offline:

```text
Subscriber ❌
```

তবুও:

```text
Topic
 ├── Message 101
 ├── Message 102
 └── Message 103
```

Subscriber ফিরে এলে consume করতে পারে—যদি configured retention ও consumer state সেই behavior support করে।

---

# 📍 8. Message ID / Index / Offset

System-কে জানতে হয়:

> **Subscriber কতদূর পর্যন্ত message consume করেছে?**

Example:

```text
Message 101
Message 102
Message 103
Message 104
Message 105
```

Subscriber:

```text
Consumed up to 103
```

Conceptually:

```text
101 ✅
102 ✅
103 ✅
104 ⏳
105 ⏳
```

এই progress message **ID, index, বা offset** দিয়ে track করা যেতে পারে।

---

# ✅ 9. Acknowledgment (ACK)

**ACK = Acknowledgment**

Subscriber যখন message সফলভাবে receive/process করেছে, তখন system-কে জানাতে পারে:

> **“আমি এই message successfully handle করেছি।”**

```text
Topic
  ↓
Message
  ↓
Subscriber
  ↓
ACK ✅
```

---

# 🔄 10. At-Least-Once Delivery

**At-least-once delivery** মানে system চেষ্টা করে message-কে অন্তত একবার deliver করতে।

```mermaid
sequenceDiagram
    participant P as 📤 Publisher
    participant T as 🟪 Topic
    participant S as 📥 Subscriber

    P->>T: Publish Message #101
    T->>S: Deliver #101
    S->>S: Process #101
    S--xT: ACK lost / connection failed
    T->>S: Retry #101
    S->>T: ACK ✅
```

---

# ♻️ 11. Duplicate Message কেন হতে পারে?

Scenario:

```text
Message #101
     ↓
Subscriber
     ↓
Process ✅
     ↓
ACK ❌
     ↓
Connection Lost
     ↓
Retry
     ↓
Message #101 Again
```

তাই:

```text
At-Least-Once
     ↓
Duplicate Possible
```

এটি distributed messaging-এর একটি গুরুত্বপূর্ণ trade-off।

---

# 🛡️ 12. Idempotency কী?

**Idempotency** হলো এমন property যেখানে একই operation multiple times execute হলেও unintended repeated effect তৈরি হয় না।

### Idempotent Example

```text
Set Status = PAID
```

বারবার execute করলেও final state:

```text
PAID
```

### Non-Idempotent Example

```text
Increase Balance by 500
```

```text
1 time  → 1000 → 1500
2 times → 1000 → 2000
```

তাই duplicate message handle করার সময় idempotency গুরুত্বপূর্ণ।

---

# 🛡️ 13. Pub/Sub + Idempotent Consumer

Subscriber message ID track করতে পারে।

```text
Message ID = PAY_123
```

তারপর:

```mermaid
flowchart TD
    M[📩 Message PAY_123] --> C{Already Processed?}

    C -->|No| P[⚙️ Process Message]
    P --> S[💾 Store Processed ID]
    S --> A[✅ ACK]

    C -->|Yes| D[🚫 Ignore Duplicate]
    D --> A

    classDef msg fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef check fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#111827;
    classDef ok fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;
    classDef dup fill:#fee2e2,stroke:#dc2626,stroke-width:3px,color:#111827;

    class M msg;
    class C check;
    class P,S,A ok;
    class D dup;
```

---

# 🔢 14. Message Ordering

Related events-এর order গুরুত্বপূর্ণ হতে পারে।

Example:

```text
M1 = Hello
M2 = How are you?
M3 = Are you free?
```

Expected:

```text
M1 → M2 → M3
```

Order বদলে:

```text
M3 → M1 → M2
```

হলে application logic বা user experience খারাপ হতে পারে।

---

# 📚 15. FIFO — First In, First Out

FIFO:

```text
First  → M1
Second → M2
Third  → M3
```

Consume:

```text
M1
 ↓
M2
 ↓
M3
```

> Exact ordering guarantee messaging technology, partitioning এবং consumer configuration-এর উপর নির্ভর করে।

---

# 💬 16. Chat Example

```mermaid
flowchart LR
    P[💬 Chat Publisher] --> T[(🟪 chat-messages)]
    T --> M1[1️⃣ Hello]
    M1 --> M2[2️⃣ How are you?]
    M2 --> M3[3️⃣ Are you free?]
    M3 --> S[📥 Subscriber]

    classDef pub fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef topic fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef msg fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#111827;
    classDef sub fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;

    class P pub;
    class T topic;
    class M1,M2,M3 msg;
    class S sub;
```

Chat-এর মতো application-এ:

```text
✅ Ordering
✅ Low Latency
✅ Duplicate Handling
✅ Reliable Delivery
```

গুরুত্বপূর্ণ হতে পারে।

---

# 🗂️ 17. Multiple Topics

বড় system-এ আলাদা event type-এর জন্য আলাদা topic রাখা যায়:

```text
order-created
payment-completed
user-signup
notification
stock-update
chat-message
```

এতে separation of concerns improve করা যায়।

```mermaid
flowchart TD
    P[📤 Publishers] --> T1[(🟪 order-created)]
    P --> T2[(🟦 payment-completed)]
    P --> T3[(🟩 notification)]
    P --> T4[(🟧 stock-update)]

    T1 --> S1[📦 Inventory]
    T2 --> S2[💳 Payment Analytics]
    T3 --> S3[🔔 Notification]
    T4 --> S4[📈 Trading Dashboard]

    classDef pub fill:#f3f4f6,stroke:#6b7280,stroke-width:3px,color:#111827;
    classDef a fill:#ede9fe,stroke:#7c3aed,stroke-width:3px,color:#111827;
    classDef b fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef c fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;
    classDef d fill:#ffedd5,stroke:#ea580c,stroke-width:3px,color:#111827;

    class P pub;
    class T1,S1 a;
    class T2,S2 b;
    class T3,S3 c;
    class T4,S4 d;
```

---

# 🔎 18. Content-Based Filtering

একটি topic-এ অনেক message থাকতে পারে।

Example:

```text
AAPL
GOOG
TSLA
MSFT
AMZN
```

একজন subscriber শুধু TSLA নিয়ে interested।

Conceptually:

```text
Topic
  ↓
Many Messages
  ↓
Filter
  ↓
Relevant Messages
```

```mermaid
flowchart LR
    T[(🟪 Stock Topic)] --> F{🔎 Filter: TSLA}
    F -->|Match| A[📈 TSLA Event]
    F -->|Ignore| B[🚫 AAPL]
    F -->|Ignore| C[🚫 GOOG]
    F -->|Ignore| D[🚫 MSFT]

    classDef topic fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef filter fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#111827;
    classDef match fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;
    classDef ignore fill:#fee2e2,stroke:#dc2626,stroke-width:2px,color:#111827;

    class T topic;
    class F filter;
    class A match;
    class B,C,D ignore;
```

---

# 🛒 19. E-Commerce Example

একটি customer order করলো:

```text
Order #123
```

Order Service event publish করলো:

```text
order-created
```

Subscribers:

```text
Inventory Service
Email Service
Analytics Service
Notification Service
```

```mermaid
flowchart TD
    U[👤 Customer] --> O[🛒 Order Service]
    O -->|📩 order-created| T[(🟪 Topic)]

    T --> I[📦 Inventory Service]
    T --> E[📧 Email Service]
    T --> A[📊 Analytics Service]
    T --> N[🔔 Notification Service]

    I --> IA[✅ Update Inventory]
    E --> EB[✉️ Send Email]
    A --> AC[📈 Record Analytics]
    N --> NA[🔔 Send Notification]

    classDef user fill:#f3f4f6,stroke:#6b7280,stroke-width:3px,color:#111827;
    classDef order fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef topic fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef service fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;

    class U user;
    class O order;
    class T topic;
    class I,E,A,N,IA,EB,AC,NA service;
```

একটি event থেকে অনেক service independently কাজ করতে পারে।

---

# 📈 20. Stock Trading Example

```text
Market Data Publisher
       ↓
 stock-price Topic
       ↓
 ┌────────┼────────┐
 ↓        ↓        ↓
Trading  Dashboard Analytics
```

Example:

```text
AAPL = 100
AAPL = 102
AAPL = 101
```

Related event-এর order গুরুত্বপূর্ণ হতে পারে।

---

# 🚨 21. Complete Failure Flow

```mermaid
sequenceDiagram
    participant P as 📤 Publisher
    participant T as 🟪 Topic
    participant S as 📥 Subscriber

    P->>T: 📩 Publish Message #101
    Note over T: Store / Retain

    T->>S: 📦 Deliver #101
    S->>S: ⚙️ Process

    S--xT: ❌ ACK lost

    Note over T: No ACK received
    T->>S: 🔄 Retry #101

    S->>S: 🛡️ Idempotent handling
    S->>T: ✅ ACK
```

এই flow থেকে মনে রাখো:

```text
No ACK
  ↓
Retry
  ↓
Duplicate Possible
  ↓
Idempotency
```

---

# 🧠 22. Pub/Sub-এর Full End-to-End Flow

```text
Publisher
    ↓
Create Event
    ↓
Create Message
    ↓
Publish to Topic
    ↓
Topic Stores Message
    ↓
Subscriber Receives Message
    ↓
Process Message
    ↓
ACK
    ↓
Consumption Progress Updated
```

---

# 🔥 23. Reliability Chain

```text
PUBLISH
   ↓
STORE
   ↓
DELIVER
   ↓
PROCESS
   ↓
ACK
```

Failure হলে:

```text
ACK Missing
   ↓
Retry
   ↓
Duplicate Possible
   ↓
Idempotent Consumer
```

---

# ⚖️ 24. Pub/Sub vs Direct Communication

| বিষয়                       | Direct Communication | Pub/Sub                   |
| -------------------------- | -------------------- | ------------------------- |
| Communication              | Service → Service    | Service → Topic → Service |
| Coupling                   | Higher               | Lower                     |
| Multiple Consumers         | Harder               | Natural fit               |
| Publisher knows consumers? | Usually yes          | Not necessarily           |
| Failure buffering          | Limited              | Topic retention can help  |
| Flexibility                | Lower                | Higher                    |
| Scalability                | More tightly coupled | Better separation         |

---

# ⚡ 25. Pub/Sub vs Basic Streaming

| বিষয়               | Basic Streaming          | Pub/Sub                      |
| ------------------ | ------------------------ | ---------------------------- |
| Main Idea          | Long-lived connection    | Topic-based messaging        |
| Intermediary       | Not necessarily          | Topic                        |
| Communication      | Direct                   | Indirect                     |
| Persistence        | Implementation-dependent | Can retain messages          |
| Multiple Consumers | Possible                 | Natural fit                  |
| Decoupling         | Lower                    | Higher                       |
| Reliability        | Implementation-dependent | ACK/retry/retention can help |

---

# 🚀 26. Kafka কোথায় আসে?

**Apache Kafka** হলো একটি distributed event streaming platform যা topic-based event/message architecture ব্যবহার করে।

Conceptually:

```text
Producer
   ↓
Kafka Topic
   ↓
Consumers
```

```mermaid
flowchart LR
    P1[📤 Order Producer] --> K[(🟪 Kafka Topic)]
    P2[📤 Payment Producer] --> K

    K --> C1[📦 Inventory Consumer]
    K --> C2[📧 Email Consumer]
    K --> C3[📊 Analytics Consumer]

    classDef producer fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef kafka fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef consumer fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;

    class P1,P2 producer;
    class K kafka;
    class C1,C2,C3 consumer;
```

Kafka-কে শুধু “simple queue” না দেখে **durable event streaming/log architecture** হিসেবেও বোঝা ভালো।

---

# 🎯 27. Interview — What is Pub/Sub?

### English

> **Publish-Subscribe is a messaging pattern where publishers send messages to topics, and subscribers consume messages from those topics without requiring direct knowledge of each other.**

### বাংলা

> **Pub/Sub হলো এমন একটি messaging pattern যেখানে publisher সরাসরি subscriber-কে message না পাঠিয়ে topic-এ publish করে, আর subscriber topic থেকে message consume করে। ফলে publisher ও subscriber loosely coupled থাকে।**

---

# 🎯 28. Interview — What is At-Least-Once Delivery?

> **At-least-once delivery means the system attempts to deliver a message at least once. If an acknowledgement is lost, the message may be retried and delivered again.**

বাংলায়:

> **System message অন্তত একবার deliver করার চেষ্টা করে। ACK হারিয়ে গেলে retry-এর কারণে একই message আবার আসতে পারে।**

---

# 🎯 29. Interview — Why is Idempotency Important?

> **Because at-least-once delivery can produce duplicate messages. Idempotency prevents duplicate processing from causing unintended repeated effects.**

---

# 🎯 30. Interview — Why use Multiple Topics?

> Different event types আলাদা রাখার জন্য, separation of concerns improve করার জন্য এবং বিভিন্ন subscriber-কে প্রয়োজনীয় data consume করার সুযোগ দেওয়ার জন্য।

---

# 🧠 31. 10-Second Explanation

কেউ যদি জিজ্ঞেস করে:

> **“Pub/Sub কী?”**

বলবে:

> **Pub/Sub হলো একটি messaging pattern যেখানে publisher message সরাসরি subscriber-কে না পাঠিয়ে topic-এ publish করে। Subscriber topic থেকে message consume করে। এতে publisher ও subscriber loosely coupled থাকে, multiple consumers support করা যায় এবং topic message retain করলে temporary failure-এর পর message consume করার সুযোগ থাকতে পারে। At-least-once delivery-এর কারণে duplicate message হতে পারে, তাই idempotency গুরুত্বপূর্ণ।**

---

# 🧠 32. Final Mental Model

```mermaid
flowchart LR
    P[📤 Publisher] -->|1️⃣ Publish| T[(🟪 Topic)]
    T -->|2️⃣ Store| M[📦 Message]
    M -->|3️⃣ Deliver| S[📥 Subscriber]
    S -->|4️⃣ Process| PR[⚙️ Processing]
    PR -->|5️⃣ ACK| T

    T -. "ACK Missing" .-> R[🔄 Retry]
    R --> S

    S --> I[🛡️ Idempotency]

    classDef pub fill:#dbeafe,stroke:#2563eb,stroke-width:3px,color:#111827;
    classDef topic fill:#ede9fe,stroke:#7c3aed,stroke-width:4px,color:#111827;
    classDef msg fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#111827;
    classDef sub fill:#dcfce7,stroke:#16a34a,stroke-width:3px,color:#111827;
    classDef retry fill:#fee2e2,stroke:#dc2626,stroke-width:3px,color:#111827;
    classDef idem fill:#ccfbf1,stroke:#0f766e,stroke-width:3px,color:#111827;

    class P pub;
    class T topic;
    class M msg;
    class S,PR sub;
    class R retry;
    class I idem;
```

---

# 🔥 33. Final Memory Trick

## Pub/Sub

```text
PUBLISHER
    ↓
  TOPIC
    ↓
 MESSAGE
    ↓
SUBSCRIBER
```

## Reliability

```text
MESSAGE
   ↓
DELIVER
   ↓
PROCESS
   ↓
ACK
```

## Failure

```text
ACK Missing
    ↓
Retry
    ↓
Duplicate Possible
    ↓
Idempotency
```

## Multiple Consumers

```text
          TOPIC
       ┌────┼────┐
       ↓    ↓    ↓
      S1   S2   S3
```

## Core Principle

> **Publisher does not need to know the Subscriber directly.**

```text
Publisher → Topic → Subscriber
```

---

# ⭐ 34. সবচেয়ে গুরুত্বপূর্ণ 10টি Point

```text
1. Pub/Sub = Publish-Subscribe messaging pattern.

2. Four key entities:
   Publisher, Subscriber, Topic, Message.

3. Publisher publishes messages to a Topic.

4. Subscriber consumes messages from a Topic.

5. Publisher and Subscriber can remain loosely coupled.

6. Topic/message retention can help during temporary failures.

7. Consumption can be tracked using IDs, indices, or offsets.

8. ACK confirms successful message handling.

9. At-Least-Once delivery can cause duplicate messages.

10. Idempotency is important for safely handling duplicates.
```

---

# 🏆 One-Line Summary

> **Pub/Sub = Publisher → Topic → Subscriber**

Reliability:

```text
Publish
   ↓
Store
   ↓
Deliver
   ↓
Process
   ↓
ACK
```

Failure:

```text
ACK Missing
   ↓
Retry
   ↓
Duplicate Possible
   ↓
Idempotency
```

> **Pub/Sub-এর আসল শক্তি হলো decoupling + scalable message distribution + reliable event handling।**

---

# 📚 Final Cheat Sheet

| Term           | Easy Meaning                                     |
| -------------- | ------------------------------------------------ |
| Publisher      | যে message publish করে                           |
| Subscriber     | যে message consume করে                           |
| Topic          | Publisher ও Subscriber-এর intermediary/channel   |
| Message        | Actual data/event                                |
| ACK            | Message processing/receipt confirmation          |
| Offset / Index | Consumer কতদূর consume করেছে তার অবস্থান         |
| At-Least-Once  | অন্তত একবার delivery করার চেষ্টা                 |
| Duplicate      | একই message একাধিকবার পাওয়া                      |
| Idempotency    | Duplicate processing-এর unintended effect ঠেকানো |
| Ordering       | Message sequence maintain করা                    |
| Persistence    | Message কিছু সময় durableভাবে রাখা                |
| Filtering      | Relevant message বেছে নেওয়া                      |
| Kafka          | Distributed event streaming platform             |

---

# 🎯 Final Rule

```text
IF many services need the same event
AND direct coupling should be minimized
→ 🟢 USE PUB/SUB

IF messages may need to survive temporary consumer failure
→ 🟢 USE MESSAGE RETENTION / PERSISTENCE

IF delivery can retry
→ 🟡 EXPECT DUPLICATES

IF duplicates are possible
→ 🛡️ DESIGN FOR IDEMPOTENCY

IF event order affects the result
→ 🔵 DESIGN FOR ORDERING
```

> **Pub/Sub = decoupled producers + topics + consumers + reliable message handling.**

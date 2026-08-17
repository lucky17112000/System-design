# 🔐 Hashing — System Design Notes

> **Hashing হলো input data-কে একটি hash value-তে convert করার process।**
> System Design-এ এই hash value ব্যবহার করে request, data বা key-কে নির্দিষ্ট server/location-এ map করা যায়।

---

## 1. Hashing কী?

সহজভাবে:

```text
Input Data
    ↓
Hash Function
    ↓
Hash Value
```

Input হতে পারে:

- Username
- Client ID
- IP Address
- String
- Number
- File
- অন্য যেকোনো hashable data

### Example

```text
hash("c1") = 11
hash("c2") = 12
hash("c3") = 13
```

এখানে:

```text
"c1" → Input
11   → Hash Value
```

---

## 2. Hash Function কী?

**Hash Function** হলো এমন একটি function যা input নিয়ে একটি hash value তৈরি করে।

```text
             Hash Function
                   │
                   ▼
Input ─────────────────────→ Hash Value
```

একই input এবং একই hash function ব্যবহার করলে একই hash value পাওয়া যায়।

```text
hash("c1") = 11
hash("c1") = 11
hash("c1") = 11
```

এই property-কে **deterministic behavior** বলা হয়।

---

## 3. Hashing কেন দরকার?

Hashing-এর একটি গুরুত্বপূর্ণ ব্যবহার হলো:

> **কোনো input-কে একটি নির্দিষ্ট location বা server-এর সাথে map করা।**

```text
Client ID
   ↓
Hash
   ↓
Hash Value
   ↓
Server Selection
```

এতে একই input বারবার একই server-এ যেতে পারে, যা caching বা data partitioning-এর মতো ক্ষেত্রে useful।

---

# 4. Load Balancer Example

ধরো আমাদের ৩টি server আছে:

```text
Server 0
Server 1
Server 2
```

এবং ৩টি client:

```text
c1
c2
c3
```

Architecture:

```text
c1 ──┐
c2 ──┼──> Load Balancer ──> Server 0
c3 ──┘                     Server 1
                           Server 2
```

এখন প্রশ্ন:

> **Load Balancer কোন request কোন server-এ পাঠাবে?**

এটাকে **Server Selection Strategy** বলা যায়।

---

# 5. Server Selection Strategy

## Random

Randomly server নির্বাচন করা:

```text
Request 1 → Server 0
Request 2 → Server 2
Request 3 → Server 1
Request 4 → Server 2
```

সমস্যা:

> একই client-এর একই request বারবার different server-এ যেতে পারে।

---

## Round Robin

Serverগুলোকে একটি নির্দিষ্ট order-এ ব্যবহার করা:

```text
Server 0 → Server 1 → Server 2 → Server 0 → ...
```

Example:

```text
Request 1 → Server 0
Request 2 → Server 1
Request 3 → Server 2
Request 4 → Server 0
```

Round Robin simple, কিন্তু cache-heavy workload-এর ক্ষেত্রে সমস্যা হতে পারে।

---

# 6. Cache কেন গুরুত্বপূর্ণ?

ধরো প্রতিটি server-এর নিজস্ব in-memory cache আছে:

```text
Server 0 → Cache 0
Server 1 → Cache 1
Server 2 → Cache 2
```

একটি request process করতে অনেক expensive computation লাগে।

প্রথমবার:

```text
Request
   ↓
Cache Check
   ↓
Cache Miss ❌
   ↓
Expensive Computation
   ↓
Response
   ↓
Cache-এ Store
```

পরেরবার একই request-এর result cache-এ থাকলে:

```text
Request
   ↓
Cache Check
   ↓
Cache Hit ✅
   ↓
Cached Response
```

এতে expensive computation আবার করতে হয় না।

---

# 7. Round Robin-এর Cache Problem

ধরো `c1` একটি request পাঠালো।

Round Robin-এর কারণে:

```text
c1
 ↓
Load Balancer
 ↓
Server 0
```

Server 0-এর cache-এ result নেই:

```text
Cache 0
   ↓
MISS ❌
```

তাই Server 0 computation করে এবং result cache-এ রাখে:

```text
Request
 ↓
Compute
 ↓
Response
 ↓
Cache 0
```

এখন:

```text
Cache 0:
Request X → Response X
```

### c1 আবার একই request পাঠালে

Round Robin অনুযায়ী এবার:

```text
c1
 ↓
Load Balancer
 ↓
Server 1
```

কিন্তু Server 1-এর cache-এ Request X নেই:

```text
Cache 1
   ↓
MISS ❌
```

ফলে Server 1 আবার expensive computation করবে।

অর্থাৎ:

> Server 0-এর cache-এ result থাকা সত্ত্বেও request Server 1-এ যাওয়ায় সেই cache ব্যবহার করা গেল না।

---

# 8. আমরা কী চাই?

আমরা চাই:

```text
Same Input
    ↓
Same Server
    ↓
Same Cache
```

প্রথম request:

```text
Request
   ↓
Cache Miss
   ↓
Compute
   ↓
Store in Cache
```

পরের request:

```text
Same Request
   ↓
Same Server
   ↓
Cache Hit ✅
   ↓
Fast Response
```

এখানেই **Hashing** useful।

---

# 9. Hashing দিয়ে Server নির্বাচন

ধরো:

```text
hash(c1) = 11
hash(c2) = 12
hash(c3) = 13
```

আমাদের server সংখ্যা:

```text
3
```

তাহলে server নির্বাচন করতে ব্যবহার করতে পারি:

```text
hash(client_id) % number_of_servers
```

অর্থাৎ:

```text
hash(client_id) % 3
```

---

# 10. Modulo (`%`) কী?

`%` হলো **Modulo Operator**।

এটি division-এর remainder দেয়।

### Example

```text
11 % 3 = 2
```

কারণ:

```text
11 ÷ 3 = 3 remainder 2
```

আর:

```text
12 % 3 = 0
13 % 3 = 1
```

---

# 11. Hash + Modulo = Server Selection

## Client c1

```text
hash(c1) = 11

11 % 3 = 2
```

তাই:

```text
c1 → Server 2
```

## Client c2

```text
hash(c2) = 12

12 % 3 = 0
```

তাই:

```text
c2 → Server 0
```

## Client c3

```text
hash(c3) = 13

13 % 3 = 1
```

তাই:

```text
c3 → Server 1
```

---

# 12. পুরো Mapping

| Client | Hash Value | Hash % 3 | Selected Server |
| ------ | ---------: | -------: | --------------- |
| c1     |         11 |        2 | Server 2        |
| c2     |         12 |        0 | Server 0        |
| c3     |         13 |        1 | Server 1        |

Diagram:

```text
c1 ─────────→ Server 2
c2 ─────────→ Server 0
c3 ─────────→ Server 1
```

---

# 13. একই Input কেন একই Server-এ যায়?

ধরো `c1` আবার request পাঠাল।

```text
hash(c1) = 11
```

আবার:

```text
11 % 3 = 2
```

তাই:

```text
c1 → Server 2
```

অর্থাৎ:

```text
Same Input
    ↓
Same Hash
    ↓
Same Server
```

এটাই hashing-based routing-এর মূল ধারণা।

---

# 14. Cache-এর জন্য Hashing কেন ভালো?

প্রথমবার:

```text
c1
 ↓
hash(c1)
 ↓
11
 ↓
11 % 3
 ↓
2
 ↓
Server 2
```

Server 2:

```text
Cache MISS
   ↓
Compute
   ↓
Store Result
```

পরেরবার:

```text
c1
 ↓
hash(c1)
 ↓
11
 ↓
11 % 3
 ↓
2
 ↓
Server 2
```

এবার:

```text
Cache HIT ✅
   ↓
Return Cached Response
```

ফলে:

- Unnecessary computation কমে
- Cache utilization বাড়ে
- Response দ্রুত হতে পারে

---

# 15. Hashing-এর Main Flow

সবচেয়ে গুরুত্বপূর্ণ flow:

```text
Input
  ↓
Hash Function
  ↓
Hash Value
  ↓
Modulo
  ↓
Server Index
  ↓
Selected Server
```

Example:

```text
c1
 ↓
hash(c1)
 ↓
11
 ↓
11 % 3
 ↓
2
 ↓
Server 2
```

---

# 16. ভালো Hash Function কেন দরকার?

আমরা চাই input-গুলোকে servers-এর মধ্যে যতটা সম্ভব ভালোভাবে distribute করতে।

ধরো:

```text
Server 0
Server 1
Server 2
```

যদি অনেক input একই server-এ যায়:

```text
           Load Balancer
          /      |               ↓       ↓       ↓
       S0      S1       S2
                ↑
                │
          অনেক request
```

তাহলে Server 1-এর উপর বেশি load পড়বে।

এটাকে **Hotspot** বলা যেতে পারে।

তাই:

> **ভালো hash function input-গুলোকে evenly distribute করতে সাহায্য করে।**

---

# 17. Hotspot কী?

যখন distribution ভালো না হয়ে অনেক request একটি নির্দিষ্ট server-এ চলে যায়:

```text
Server 0 → 100 requests
Server 1 → 9000 requests
Server 2 → 100 requests
```

তখন:

```text
Server 1
   ↓
Hotspot
   ↓
High Load
```

এটি system performance-এর জন্য সমস্যা তৈরি করতে পারে।

---

# 18. Hash Function নিজে কি Server নির্বাচন করে?

**না।**

Hash Function প্রথমে hash value তৈরি করে:

```text
c1
 ↓
Hash Function
 ↓
11
```

তারপর routing logic ব্যবহার করে:

```text
11 % 3 = 2
```

তারপর:

```text
2 → Server 2
```

সুতরাং:

```text
Input
 ↓
Hash Function
 ↓
Hash Value
 ↓
Routing Calculation
 ↓
Server
```

---

# 19. Hashing কি শুধু Load Balancer-এর জন্য?

না।

Hashing বিভিন্ন system এবং computer science problem-এ ব্যবহার করা হয়:

- Load Balancing
- Hash Tables
- Caching
- Distributed Systems
- Data Partitioning
- Deduplication
- Password-related systems
- Content Identification

তবে **সব ক্ষেত্রে একই type-এর hash function ব্যবহার করা হয় না**।

---

# 20. Hashing-এর Important Properties

## 20.1 Deterministic

একই input → একই hash value

```text
hash("c1") = 11
hash("c1") = 11
hash("c1") = 11
```

## 20.2 Fixed-Size Output

Hash function একটি নির্দিষ্ট ধরনের output দেয়, যদিও input-এর size ছোট বা বড় হতে পারে।

```text
Input
   ↓
Hash Function
   ↓
Hash Value
```

## 20.3 Good Distribution

ভালো hashing input-গুলোকে output space-এ ভালোভাবে ছড়িয়ে দিতে সাহায্য করে।

## 20.4 Fast

Load balancing, caching বা data partitioning-এর ক্ষেত্রে hash calculation সাধারণত দ্রুত হওয়া desirable।

---

# 21. Hash কি সবসময় Unique?

**না।**

Different input-এর একই hash value হতে পারে।

Example:

```text
hash(A) = 10
hash(B) = 10
```

এটাকে বলে:

> **Hash Collision**

Diagram:

```text
Input A ──┐
          ├──→ Same Hash Value
Input B ──┘
```

তাই:

> **Hashing মানে প্রতিটি input-এর জন্য guaranteed unique value পাওয়া নয়।**

---

# 22. Hashing বনাম Encryption

এগুলো গুলিয়ে ফেলবে না।

## Hashing

```text
Input
  ↓
Hash
```

Hashing সাধারণত one-way transformation হিসেবে ব্যবহৃত হয়।

## Encryption

```text
Plaintext
   ↓
Encryption
   ↓
Ciphertext
```

Encryption key-এর মাধ্যমে data decrypt করা সম্ভব।

সহজভাবে:

> **Hashing = Data-এর fingerprint-এর মতো**

> **Encryption = Data-কে lock করার মতো**

---

# 23. Cryptographic Hash বনাম Password Hashing

সব hash function একই উদ্দেশ্যে তৈরি করা হয় না।

### Cryptographic Hash Examples

- MD5
- SHA-1
- SHA-256

### Password Hashing

Password-এর জন্য বিশেষ password hashing algorithms ব্যবহার করা হয়:

- bcrypt
- scrypt
- Argon2

> **bcrypt-এর মতো password hashing algorithm-কে general-purpose load balancing hash function হিসেবে ভাবা ঠিক নয়।**

Load balancing বা partitioning-এর ক্ষেত্রে সাধারণত দ্রুত এবং ভালোভাবে distributed hash function প্রয়োজন হয়।

---

# 24. Real-Life Analogy: Locker

ধরো একটি school-এ 10টি locker আছে:

```text
Locker 0
Locker 1
Locker 2
...
Locker 9
```

Student ID:

```text
1023
```

আমরা করি:

```text
1023 % 10 = 3
```

তাহলে:

```text
Student 1023 → Locker 3
```

Student 1023 আবার এলে:

```text
1023 % 10 = 3
```

আবার:

```text
Locker 3
```

এখানে **input → calculation → fixed location** ধারণাটি hashing-based mapping বোঝার জন্য useful।

---

# 25. System Design Example

ধরো:

```text
1000 Clients
      ↓
Load Balancer
      ↓
Hash(Client ID)
      ↓
Modulo
      ↓
Server Selection
```

Conceptually:

```text
Client ID
   ↓
Hash
   ↓
Integer
   ↓
% Number of Servers
   ↓
Server 0 / 1 / 2 / ...
```

এতে একই client identifier বারবার একই server-এ map হতে পারে, যতক্ষণ একই mapping rules ব্যবহার করা হচ্ছে।

---

# 26. Important Limitation of Simple Modulo Hashing

Simple modulo-based mapping-এ server সংখ্যা পরিবর্তন করলে mapping অনেকটাই বদলে যেতে পারে।

উদাহরণ:

```text
Before:
3 Servers
```

```text
hash(key) % 3
```

পরে:

```text
4 Servers
```

এখন:

```text
hash(key) % 4
```

করলে অনেক key-এর selected server বদলে যেতে পারে।

> **এই chapter-এ Consistent Hashing explain করা হচ্ছে না।**

---

# 27. Interview Answer — English

> **Hashing is a technique that converts an input such as a client ID, username, or key into a hash value using a hash function. In system design, this hash value can be used to deterministically map requests or data to a specific server or partition. For example, with three servers, we can use `hash(clientId) % 3` to select a server. This can make repeated requests from the same client map to the same server, improving cache utilization and reducing unnecessary recomputation. A good hash function should distribute inputs evenly to avoid hotspots.**

---

# 28. Interview Answer — বাংলা

> **Hashing হলো এমন একটি process যেখানে client ID, username বা key-এর মতো input-কে একটি hash function-এর মাধ্যমে একটি hash value-তে convert করা হয়। System Design-এ এই hash value ব্যবহার করে request বা data-কে নির্দিষ্ট server বা partition-এ map করা যায়। যেমন ৩টি server থাকলে `hash(clientId) % 3` ব্যবহার করে server নির্বাচন করা যায়। ফলে একই client-এর request একই server-এ যেতে পারে, যার কারণে cache reuse করা যায় এবং unnecessary computation কমানো সম্ভব হয়। একটি ভালো hash function input-গুলোকে evenly distribute করে যাতে কোনো একটি server-এর উপর অতিরিক্ত load না পড়ে।**

---

# 29. 10-Second Explanation

কেউ যদি জিজ্ঞেস করে:

> **"Hashing কী?"**

বলবে:

> **Hashing হলো input data-কে একটি hash value-তে convert করার process। এই hash value ব্যবহার করে system data বা request-কে একটি নির্দিষ্ট location বা server-এর সাথে map করতে পারে। যেমন ৩টি server থাকলে `hash(key) % 3` ব্যবহার করে server নির্বাচন করা যায়।**

---

# 30. Final Mental Model

```text
                  Client Request
                        │
                        ▼
                  Load Balancer
                        │
                  Client ID / Key
                        │
                        ▼
                  Hash Function
                        │
                        ▼
                    Hash Value
                        │
                        ▼
                  % #Servers
                        │
                        ▼
                   Server Index
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Server 0   Server 1   Server 2
```

Example:

```text
c1
 ↓
hash(c1) = 11
 ↓
11 % 3 = 2
 ↓
Server 2
```

---

# 🧠 Final Memory Trick

## Hashing

```text
INPUT
  ↓
HASH
  ↓
HASH VALUE
  ↓
MAPPING
```

## Load Balancing Example

```text
Client ID
   ↓
Hash
   ↓
Hash Value
   ↓
% Number of Servers
   ↓
Server
```

### সবচেয়ে গুরুত্বপূর্ণ ৫টি point

```text
1. Hashing converts input → hash value
2. Same input → same hash with the same function
3. hash + modulo can be used for server selection
4. Good distribution helps avoid hotspots
5. Hashing is different from encryption
```

> **এক লাইনে:**  
> **Hashing = Input → Hash Value → Mapping**

> **Load Balancer-এর ক্ষেত্রে:**  
> **Client ID → Hash → Modulo → Server**

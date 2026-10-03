# ⚡ Caching — System Design Notes

> **Caching হলো frequently বা recently ব্যবহৃত data-কে একটি দ্রুত-access location-এ temporarily store করে রাখা, যাতে পরের বার একই data চাইলে original source থেকে আবার fetch বা compute করতে না হয়।**

> System Design-এ caching ব্যবহার করা হয় **latency কমাতে, database load কমাতে এবং expensive computation এড়াতে।**

---

# 1. Caching কী?

সহজভাবে:

```text
Original Source
      ↓
   Data Fetch
      ↓
    Cache
      ↓
  Fast Access
```

Cache হলো একটি **temporary storage layer** যেখানে frequently accessed data রাখা হয়।

ধরো:

```text
User → Server → Database
```

প্রতিবার database থেকে data আনতে গেলে database-এর উপর load বাড়ে।

তাই আমরা cache যোগ করতে পারি:

```text
User
 ↓
Server
 ↓
Cache
 ↓
Database
```

প্রথমবার data database থেকে আসবে এবং cache-এ রাখা হবে।

পরেরবার একই data চাইলে:

```text
User
 ↓
Server
 ↓
Cache ✅
 ↓
Response
```

Database-এ যেতে হবে না।

---

# 2. Real-Life Example

ধরো তুমি প্রতিদিন একটি বই পড়ো।

বইটি library-তে আছে:

```text
You → Library → Book
```

প্রতিদিন library-তে গিয়ে বই আনা inconvenient।

তাই তুমি বইটি নিজের table-এ রেখে দিলে:

```text
You → Table → Book
```

এখন বই দরকার হলে table থেকেই নিয়ে পড়তে পারো।

এখানে:

```text
Library = Database
Table    = Cache
You      = Application/User
```

অর্থাৎ:

> **Cache হলো frequently needed data-কে original source-এর তুলনায় সহজে ও দ্রুত access করার জন্য কাছাকাছি রাখা।**

---

# 3. Cache কেন দরকার?

Caching-এর প্রধান উদ্দেশ্য হলো:

```text
1. Response দ্রুত করা
2. Database load কমানো
3. Network calls কমানো
4. Expensive computation কমানো
5. System scalability improve করা
```

Without Cache:

```text
Request
   ↓
Server
   ↓
Database
   ↓
Response
```

With Cache:

```text
Request
   ↓
Server
   ↓
Cache
   ↓
Fast Response
```

---

# 4. Cache কোথায় থাকতে পারে?

Cache system-এর বিভিন্ন level-এ থাকতে পারে।

```text
Client
  ↓
Server
  ↓
Cache
  ↓
Database
```

প্রধানত:

```text
1. Client-side Cache
2. Server-side / Application Cache
3. Distributed Cache
4. Database-এর সামনে Cache
```

---

# 5. Client-Side Cache

Browser বা client device-এর মধ্যেও cache থাকতে পারে।

Example:

```text
Browser
 ↓
Cached Image
Cached CSS
Cached JavaScript
Cached Files
```

ধরো তুমি একটি website প্রথমবার visit করলে।

Browser কিছু resource locally store করলো।

পরেরবার website visit করলে সব resource নতুন করে download করতে হয় না।

ফলে:

```text
Less Network Request
        ↓
Faster Loading
```

---

# 6. Server-Side Cache

Application server নিজের memory-তেও data cache করতে পারে।

```text
User
 ↓
Application Server
 ↓
Local Cache
```

Example:

ধরো homepage-এর একই content হাজার হাজার user দেখছে।

প্রতিবার database query না করে server cached result return করতে পারে।

---

# 7. Distributed Cache

বড় system-এ একাধিক application server থাকতে পারে।

```text
                 Load Balancer
                       ↓
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Server 1      Server 2      Server 3
          └────────────┼────────────┘
                       ↓
                 Distributed Cache
                       ↓
                    Database
```

এখানে Redis-এর মতো shared cache ব্যবহার করা যেতে পারে।

সব application server একই cache access করতে পারে।

---

# 8. Database-এর সামনে Cache

System Design-এ খুব common architecture:

```text
Client
   ↓
Backend Server
   ↓
Redis / Cache
   ↓
PostgreSQL / MySQL
```

Request প্রথমে cache-এ যায়।

Data পাওয়া গেলে database query করার প্রয়োজন হয় না।

---

# 9. Cache Hit

যখন requested data cache-এর মধ্যে পাওয়া যায়:

```text
Request
   ↓
Cache
   ↓
Data Found ✅
   ↓
Response
```

এটাকে বলে:

> **Cache Hit**

Example:

```text
GET /users/101

Cache:
user:101 → Rahim
```

তাহলে:

```text
Cache → HIT ✅
```

এবং cached data return করা হবে।

---

# 10. Cache Miss

যখন requested data cache-এ পাওয়া যায় না:

```text
Request
   ↓
Cache
   ↓
Data Not Found ❌
   ↓
Database
```

এটাকে বলে:

> **Cache Miss**

Example:

```text
GET /users/999

Cache:
user:999 → নেই
```

তখন database থেকে data fetch করতে হবে।

---

# 11. Cache Hit vs Cache Miss

| বিষয়              | Cache Hit       | Cache Miss    |
| ----------------- | --------------- | ------------- |
| Data cache-এ আছে? | Yes ✅          | No ❌         |
| Database call     | সাধারণত লাগে না | লাগে          |
| Response          | দ্রুত           | তুলনামূলক ধীর |
| DB load           | কম              | বেশি          |

---

# 12. Cache Hit Ratio

System Design-এ একটি important metric হলো:

> **Cache Hit Ratio**

Formula:

```text
Cache Hit Ratio =
Cache Hits / Total Requests
```

Example:

```text
Total Requests = 1000
Cache Hits     = 900
```

তাহলে:

```text
Hit Ratio = 900 / 1000
          = 90%
```

অর্থাৎ 90% request cache থেকেই serve হয়েছে।

Higher hit ratio সাধারণত cache-এর effectiveness বোঝাতে সাহায্য করে।

---

# 13. Cache শুধু Database-এর জন্য নয়

Caching-এর আরেকটি গুরুত্বপূর্ণ use case:

> **Expensive Computation**

ধরো একটি report তৈরি করতে 5 seconds লাগে।

```text
Request
  ↓
Large Dataset
  ↓
Complex Calculation
  ↓
Report
```

একই report 1000 বার request হলে:

```text
User 1 → Compute
User 2 → Compute
User 3 → Compute
...
User 1000 → Compute
```

এটা inefficient।

তাই প্রথমবার result calculate করে cache করা যায়।

```text
Request
   ↓
Cache
   ↓
MISS
   ↓
Expensive Computation
   ↓
Store Result in Cache
```

পরেরবার:

```text
Request
   ↓
Cache
   ↓
HIT ✅
   ↓
Cached Result
```

ফলে একই expensive computation বারবার করতে হয় না।

---

# 14. Example — Expensive Computation

ধরো:

```text
Monthly Sales Report
```

Generate করতে:

```text
5 Million Records
       ↓
Filtering
       ↓
Grouping
       ↓
Aggregation
       ↓
Calculation
       ↓
Final Report
```

Calculation অনেক expensive।

তাই:

```text
First Request
     ↓
Calculate
     ↓
Cache Result
```

পরবর্তী request:

```text
Request
   ↓
Cache
   ↓
Return Result ✅
```

---

# 15. Typical Cache Read Flow

একটি common read flow:

```text
Request
   ↓
Check Cache
   ↓
 ┌───────────────┐
 │ Data Exists?  │
 └───────┬───────┘
     Yes │ No
         │
         ▼
      Cache Hit
         │
         ▼
      Response
```

Cache miss হলে:

```text
Request
   ↓
Check Cache
   ↓
Cache Miss ❌
   ↓
Database
   ↓
Get Data
   ↓
Store in Cache
   ↓
Response
```

---

# 16. Cache-Aside Pattern

একটি খুব common caching strategy হলো:

> **Cache-Aside**

এখানে application নিজেই cache manage করে।

Read flow:

```text
Request
   ↓
Check Cache
   ↓
 ┌─────────┴─────────┐
 ↓                   ↓
HIT                 MISS
 ↓                   ↓
Return              Database
                         ↓
                    Put in Cache
                         ↓
                      Return
```

---

# 17. Cache-Aside Example

ধরো:

```text
GET /users/101
```

### First Request

```text
Server
  ↓
Cache
  ↓
MISS ❌
  ↓
Database
  ↓
User Data
  ↓
Cache
  ↓
Response
```

### Second Request

```text
Server
  ↓
Cache
  ↓
HIT ✅
  ↓
Response
```

এতে database query কমে যায়।

---

# 18. Data Update হলে Problem কেন হয়?

Caching-এর সবচেয়ে গুরুত্বপূর্ণ problem হলো:

> **Consistency**

ধরো একই data দুই জায়গায় আছে:

```text
Database:
Age = 25

Cache:
Age = 25
```

এখন user age update করলো:

```text
Age = 26
```

Database update হলো:

```text
Database:
Age = 26
```

কিন্তু cache এখনও:

```text
Cache:
Age = 25
```

তাহলে:

```text
Database ≠ Cache
```

এখানে cache-এর data হলো:

> **Stale Data**

---

# 19. Stale Data কী?

Stale data হলো এমন data যা cache-এ আছে কিন্তু original source-এর latest data নয়।

Example:

```text
Database → Price = 1200
Cache    → Price = 1000 ❌
```

Cache পুরোনো data return করছে।

এটাই stale cache problem।

---

# 20. Two Sources of Truth

যখন একই data:

```text
Database
+
Cache
```

দুই জায়গায় থাকে, তখন consistency maintain করতে হয়।

Example:

```text
Database → Age = 26
Cache    → Age = 25
```

এখন question:

> কোন value correct?

System Design-এ সাধারণত database-কে authoritative/source data হিসেবে treat করা হয়, আর cache-কে তার দ্রুত-access copy হিসেবে ধরা হয়।

---

# 21. Write-Through Cache

একটি গুরুত্বপূর্ণ strategy:

> **Write-through caching**

এখানে write operation-এর সময় cache এবং database—দুই জায়গায় update করা হয়।

```text
User
 ↓
Application
 ↓
 ┌──────────────┐
 ↓              ↓
Cache          Database
 ↓              ↓
Updated        Updated
```

Example:

```text
Old:
Cache = 25
DB    = 25
```

User:

```text
Age = 26
```

তারপর:

```text
Cache = 26
Database = 26
```

দুই জায়গাই update করার চেষ্টা করা হয়।

---

# 22. Write-Through Example — Product Stock

ধরো:

```text
Product Stock = 20
```

Cache:

```text
product:5001 → stock = 20
```

Database:

```text
stock = 20
```

একজন product কিনলো।

নতুন stock:

```text
19
```

Write-through:

```text
Purchase
   ↓
Application
   ├────────────→ Cache = 19
   │
   └────────────→ Database = 19
```

ফলে:

```text
Cache = 19
Database = 19
```

---

# 23. Write-Through-এর Advantage

```text
1. Cache এবং DB একই value রাখার চেষ্টা করে
2. Stale data কম হতে পারে
3. Read-এর সময় latest cached data পাওয়া যেতে পারে
4. Data consistency তুলনামূলকভাবে সহজে maintain করা যায়
```

---

# 24. Write-Through-এর Limitation

Write operation-এ দুই জায়গায় update করতে হয়:

```text
Cache
+
Database
```

তাই write path-এ অতিরিক্ত work থাকে।

আর distributed systems-এ failure হলে সমস্যা হতে পারে।

Example:

```text
Cache Update ✅
Database Update ❌
```

তখন আবার inconsistency তৈরি হতে পারে।

---

# 25. Write-Back Cache

আরেকটি strategy:

> **Write-back caching**

এখানে প্রথমে cache update করা হয়।

Database পরে asynchronousভাবে update করা হয়।

```text
User
 ↓
Application
 ↓
Cache
 ↓
Immediate Response
 ↓
Async Database Update
```

---

# 26. Write-Back Example

ধরো:

```text
Database = 100
Cache    = 100
```

User update করলো:

```text
120
```

Write-back:

```text
User
 ↓
Cache = 120
 ↓
Response
```

Database তখন:

```text
Database = 100
```

পরে background process:

```text
Cache
 ↓
Database
```

তারপর:

```text
Cache = 120
Database = 120
```

---

# 27. Write-Back কেন Fast?

কারণ user request-কে database write শেষ হওয়া পর্যন্ত wait করতে নাও হতে পারে।

```text
User
 ↓
Cache Update
 ↓
Response ✅
```

Database update:

```text
Background Process
       ↓
    Database
```

তাই write-heavy workload-এ latency কমানো যেতে পারে।

---

# 28. Write-Back-এর Risk

সবচেয়ে গুরুত্বপূর্ণ risk:

> **Cache failure হওয়ার আগে database-এ data persist না হলে data loss হতে পারে।**

Example:

```text
Database = 100
Cache    = 120
```

এখন cache crash করলো:

```text
Cache ❌
```

Database এখনও:

```text
100
```

তাহলে `120` update হারিয়ে যেতে পারে, যদি অন্য কোনো durability mechanism না থাকে।

---

# 29. Write-Through vs Write-Back

| বিষয়                 | Write-Through             | Write-Back                 |
| -------------------- | ------------------------- | -------------------------- |
| Cache update         | Immediately               | Immediately                |
| DB update            | Write path-এ              | পরে async                  |
| Write latency        | তুলনামূলক বেশি হতে পারে   | কম হতে পারে                |
| Consistency          | তুলনামূলক সহজ             | কিছু সময় mismatch হতে পারে |
| Data loss risk       | তুলনামূলক কম              | বেশি হতে পারে              |
| Write-heavy workload | ভালো                      | Useful                     |
| Implementation       | তুলনামূলক straightforward | More complex               |

### Memory Trick

```text
Write-Through
= Write to Cache + DB

Write-Back
= Write Cache First
  DB Later
```

---

# 30. Cache Invalidation

Caching-এর সবচেয়ে famous problem:

> **Cache Invalidation**

মানে পুরোনো/invalid cache data remove বা update করা।

ধরো:

```text
Cache:
Product Price = 500
```

Database update হলো:

```text
Product Price = 550
```

তাহলে cache-কে:

```text
500 ❌
```

থেকে:

```text
550 ✅
```

করতে হবে অথবা cache entry delete করতে হবে।

---

# 31. Cache Invalidation Example

একটি common approach:

```text
Update Database
       ↓
Invalidate Cache
```

Example:

```text
Update User
     ↓
Database = 26
     ↓
Delete user:101 from Cache
```

পরের request:

```text
Cache MISS
   ↓
Database
   ↓
Age = 26
   ↓
Store in Cache
```

---

# 32. Cache Invalidation Strategies

সাধারণভাবে:

```text
1. Update Cache
2. Delete/Invalidate Cache
3. Use TTL
4. Write-Through
5. Write-Back
```

কোন strategy ব্যবহার হবে তা workload এবং consistency requirement-এর ওপর নির্ভর করে।

---

# 33. TTL — Time To Live

> **TTL = Time To Live**

মানে একটি cache entry কতক্ষণ valid থাকবে।

Example:

```text
user:101

TTL = 60 seconds
```

60 seconds পরে entry expire করবে।

```text
Cache
 ↓
Expired ❌
```

তারপর নতুন request database থেকে fresh data fetch করতে পারে।

---

# 34. TTL Example

ধরো:

```text
News Data
TTL = 30 seconds
```

Flow:

```text
Request
 ↓
Cache HIT
 ↓
Return
```

30 seconds পরে:

```text
Cache Entry
 ↓
Expired
```

Next request:

```text
Cache MISS
 ↓
Database
 ↓
Fresh Data
 ↓
Cache
```

---

# 35. TTL-এর সুবিধা

```text
1. Stale data-এর lifetime কমানো যায়
2. Cache automatically expire হতে পারে
3. Manual invalidation-এর dependency কিছুটা কমে
4. Temporary data-এর জন্য useful
```

---

# 36. TTL-এর Problem

TTL বেশি হলে:

```text
Fresh DB
   ≠
Stale Cache
```

অনেকক্ষণ পুরোনো data থাকতে পারে।

TTL খুব কম হলে:

```text
Frequent Expiration
      ↓
More Cache Miss
      ↓
More DB Requests
```

তাই TTL carefully choose করতে হয়।

---

# 37. Cache Eviction

Cache memory limited।

ধরো:

```text
Cache Capacity = 10 GB
Data = 20 GB
```

সব data রাখা সম্ভব নয়।

তাই কিছু entries remove করতে হবে।

এটাকে বলে:

> **Cache Eviction**

---

# 38. Common Eviction Policies

### LRU

> **Least Recently Used**

যে data অনেকদিন ব্যবহার হয়নি, সেটি আগে remove করা হয়।

```text
A → Recently Used
B → Recently Used
C → Not Used for long time

Evict → C
```

---

### LFU

> **Least Frequently Used**

যে data সবচেয়ে কমবার ব্যবহৃত হয়েছে, সেটি আগে remove করা হয়।

```text
A → Used 100 times
B → Used 20 times
C → Used 2 times

Evict → C
```

---

### FIFO

> **First In, First Out**

যে entry সবচেয়ে আগে cache-এ এসেছে, সেটি আগে remove করা হয়।

```text
A → First
B
C → Latest

Evict → A
```

---

# 39. Local Cache vs Distributed Cache

## Local Cache

প্রতিটি server-এর নিজের cache থাকে।

```text
Server 1 → Cache 1
Server 2 → Cache 2
Server 3 → Cache 3
```

Problem:

```text
Cache 1 → Age 26
Cache 2 → Age 25
```

Different server different value রাখতে পারে।

---

## Distributed Cache

একটি shared cache থাকে।

```text
Server 1 ─┐
Server 2 ─┼──→ Redis
Server 3 ─┘
```

সব server একই cache ব্যবহার করতে পারে।

---

# 40. Redis কেন Cache হিসেবে জনপ্রিয়?

Redis একটি in-memory data store।

Conceptually:

```text
Application
     ↓
Redis
     ↓
Database
```

RAM-based access হওয়ার কারণে এটি database-এর তুলনায় খুব দ্রুত cached data access করতে পারে।

Redis সাধারণত ব্যবহার করা হয়:

```text
Caching
Session Storage
Counters
Rate Limiting
Queues
Distributed Coordination
```

> এখানে গুরুত্বপূর্ণ বিষয় হলো: Redis শুধু cache-এর জন্যই ব্যবহৃত হয় না।

---

# 41. Cache কি Database-এর Replacement?

**না।**

সাধারণভাবে:

```text
Database = Durable / Persistent Data
Cache    = Fast Temporary Copy
```

Database-এর উদ্দেশ্য:

```text
Long-term Storage
Persistence
Source Data
```

Cache-এর উদ্দেশ্য:

```text
Fast Access
Lower Latency
Reduce DB Load
```

---

# 42. Cache Failure হলে কী হবে?

System Design-এ cache failure-এর কথা অবশ্যই ভাবতে হবে।

ধরো:

```text
Application
    ↓
Redis ❌
```

তখন application database-এ fallback করতে পারে, depending on system design।

```text
Application
   ↓
Cache ❌
   ↓
Database
   ↓
Response
```

কিন্তু সব request database-এ গেলে:

```text
Traffic
  ↓
Database
  ↓
High Load
```

তাই cache failure handling গুরুত্বপূর্ণ।

---

# 43. Cache Stampede

ধরো একটি popular cache entry একই সময়ে expire হয়ে গেল।

```text
Cache Entry → Expired
```

হাজার হাজার request একসাথে database-এ গেল:

```text
1000 Requests
      ↓
   Cache MISS
      ↓
  1000 DB Queries
```

এতে database-এর উপর huge load পড়ে।

এ ধরনের situation-কে সাধারণভাবে বলা হয়:

> **Cache Stampede / Thundering Herd**

সমাধানের জন্য locking, request coalescing, staggered expiration, refresh-ahead ইত্যাদি technique ব্যবহার করা যেতে পারে।

---

# 44. Cache Penetration

ধরো user এমন একটি key request করলো যার data database-এও নেই।

```text
Request
  ↓
Cache MISS
  ↓
Database
  ↓
Data Not Found
```

একই invalid key বারবার request করলে:

```text
Request
 ↓
Cache MISS
 ↓
DB
 ↓
NOT FOUND

Request
 ↓
Cache MISS
 ↓
DB
 ↓
NOT FOUND
```

এতে database unnecessary load নিতে পারে।

এ ধরনের problem-কে বলা হয়:

> **Cache Penetration**

একটি possible technique হলো negative caching।

---

# 45. Cache Avalanche

যখন অনেক cache entry একই সময় বা কাছাকাছি সময়ে expire হয়ে যায়:

```text
Many Cache Entries
       ↓
Expire Together
       ↓
Many Cache Misses
       ↓
Database Overload
```

এটাকে বলা হয়:

> **Cache Avalanche**

একটি mitigation technique হলো TTL-এ randomness/jitter ব্যবহার করা, যাতে সব entry একই সময়ে expire না করে।

---

# 46. Cache Key

Cache-এ data সাধারণত key দিয়ে store করা হয়।

Example:

```text
Key:
user:101

Value:
{
  "name": "Rahim",
  "age": 25
}
```

আরেকটি:

```text
product:5001
```

Value:

```text
{
  "name": "Laptop",
  "price": 80000
}
```

Good cache key design খুব important।

---

# 47. Cache Key Naming

সাধারণ pattern:

```text
entity:id
```

Example:

```text
user:101
product:5001
order:9001
```

আর complex key:

```text
user:101:profile
product:5001:details
```

এতে key identify করা সহজ হয়।

---

# 48. What Should We Cache?

সাধারণত cache করা ভালো যখন:

```text
Frequently Read
+
Relatively Expensive to Fetch/Compute
+
Does Not Change Every Second
```

Example:

```text
Popular Products
User Profiles
Popular Posts
Search Results
Expensive Reports
Configuration Data
```

---

# 49. What Should We Be Careful About Caching?

যে data খুব frequently change হয় বা stale value dangerous হতে পারে সেখানে সতর্ক থাকতে হবে।

Example:

```text
Highly Real-Time Data
Sensitive Transactions
Rapidly Changing Inventory
```

কোন data cache হবে সেটা application-এর consistency requirement-এর ওপর নির্ভর করে।

---

# 50. Complete E-Commerce Example

ধরো একটি e-commerce application:

```text
                    Client
                       ↓
                 Load Balancer
                       ↓
                 Backend Server
                       ↓
                   Redis
                       ↓
                 PostgreSQL
```

User request:

```text
GET /products/500
```

---

## Case 1 — Cache Hit

```text
Client
  ↓
Backend
  ↓
Redis
  ↓
HIT ✅
  ↓
Response
```

Database query নেই।

---

## Case 2 — Cache Miss

```text
Client
  ↓
Backend
  ↓
Redis
  ↓
MISS ❌
  ↓
PostgreSQL
  ↓
Product Data
  ↓
Redis
  ↓
Response
```

---

## Case 3 — Product Update

Old:

```text
Cache = 1000
DB    = 1000
```

Price changed:

```text
1200
```

Write-through:

```text
Cache = 1200
DB    = 1200
```

Cache-aside invalidation:

```text
DB = 1200
   ↓
Delete Cache Entry
   ↓
Next Request
   ↓
Cache MISS
   ↓
DB = 1200
   ↓
Cache = 1200
```

---

# 51. Caching vs Direct Database Access

## Without Cache

```text
1000 Requests
      ↓
1000 DB Queries
```

## With Cache

ধরো:

```text
1000 Requests
      ↓
900 Cache Hits
100 Cache Misses
```

Conceptually:

```text
900 Requests → Cache
100 Requests → Database
```

ফলে database load অনেক কমতে পারে।

> Actual performance depends on workload, cache hit rate, network latency, data size, and implementation.

---

# 52. Caching-এর Main Advantages

```text
✅ Lower Latency
✅ Faster Response
✅ Reduced Database Load
✅ Reduced Network Calls
✅ Avoid Repeated Computation
✅ Better Scalability
✅ Better User Experience
```

---

# 53. Caching-এর Main Disadvantages

```text
❌ Stale Data
❌ Cache Invalidation Complexity
❌ Memory Cost
❌ Cache Failure
❌ Data Consistency Problems
❌ Eviction Management
❌ More System Complexity
```

---

# 54. The Main Trade-off

Caching-এর সবচেয়ে important idea:

```text
More Performance
       ↕
More Complexity
```

Cache system performance improve করে:

```text
Latency ↓
DB Load ↓
Repeated Work ↓
```

কিন্তু complexity বাড়ায়:

```text
Consistency
Invalidation
Expiration
Failure Handling
Eviction
```

তাই:

> **Cache is not free. It improves performance at the cost of additional system complexity.**

---

# 55. Full Caching Flow

একটি complete mental model:

```text
                     User Request
                           │
                           ▼
                      Application
                           │
                           ▼
                     Check Cache
                           │
                  ┌────────┴────────┐
                  │                 │
                HIT               MISS
                  │                 │
                  ▼                 ▼
              Return            Database
              Cached Data           │
                                    ▼
                              Get Fresh Data
                                    │
                                    ▼
                               Store in Cache
                                    │
                                    ▼
                                 Return
```

---

# 56. Data Update Flow

### Write-Through

```text
                 WRITE
                   │
          ┌────────┴────────┐
          ▼                 ▼
        Cache           Database
          │                 │
          ▼                 ▼
       Updated            Updated
```

### Write-Back

```text
                  WRITE
                    │
                    ▼
                  Cache
                    │
                    ▼
              Fast Response
                    │
                    ▼
             Async DB Update
                    │
                    ▼
                Database
```

### Cache-Aside

```text
READ:

Cache
 ├── HIT  → Return
 │
 └── MISS → Database
              ↓
          Put in Cache
              ↓
            Return
```

---

# 57. Cache Strategy Comparison

| Strategy      | Read        | Write                 | Main Benefit           | Main Concern               |
| ------------- | ----------- | --------------------- | ---------------------- | -------------------------- |
| Cache-Aside   | Cache first | App manages cache/DB  | Simple & flexible      | Invalidation               |
| Write-Through | Cache       | Cache + DB            | Better synchronization | Write latency              |
| Write-Back    | Cache       | Cache first, DB later | Faster writes          | Data loss/consistency risk |

---

# 58. Important Cache Terms

| Term              | Meaning                                      |
| ----------------- | -------------------------------------------- |
| Cache             | Fast temporary data storage                  |
| Cache Hit         | Data found in cache                          |
| Cache Miss        | Data not found in cache                      |
| Stale Data        | Old/expired information                      |
| TTL               | Time To Live                                 |
| Eviction          | Removing cache entries                       |
| Invalidation      | Making old cache data invalid                |
| LRU               | Least Recently Used                          |
| LFU               | Least Frequently Used                        |
| Cache Stampede    | Many requests hit DB after cache miss/expiry |
| Cache Penetration | Repeated requests for non-existent data      |
| Cache Avalanche   | Many entries expire together                 |

---

# 59. Interview Question — What is Caching?

### English Answer

> **Caching is the process of storing frequently or recently accessed data in a faster storage layer so that future requests can be served with lower latency without repeatedly accessing the original data source. In system design, caching helps reduce database load, network calls, and expensive computations.**

### বাংলা Answer

> **Caching হলো frequently বা recently accessed data-কে একটি দ্রুত storage layer-এ temporarily store করে রাখার process, যাতে পরবর্তী request-এ original source থেকে আবার data fetch বা computation করতে না হয়। System Design-এ caching latency কমায়, database load কমায় এবং expensive computation avoid করতে সাহায্য করে।**

---

# 60. Interview Question — What is Cache Hit?

> **Cache Hit হলো যখন requested data cache-এর মধ্যে পাওয়া যায় এবং cache থেকেই response return করা সম্ভব হয়।**

```text
Request
 ↓
Cache
 ↓
HIT ✅
 ↓
Response
```

---

# 61. Interview Question — What is Cache Miss?

> **Cache Miss হলো যখন requested data cache-এ পাওয়া যায় না এবং application-কে original source, যেমন database, থেকে data fetch করতে হয়।**

```text
Request
 ↓
Cache
 ↓
MISS ❌
 ↓
Database
```

---

# 62. Interview Question — Write-Through vs Write-Back

### Write-Through

> Data write করার সময় cache এবং database দুটোতেই update করা হয়।

```text
Write
 ↓
Cache + DB
```

### Write-Back

> প্রথমে cache update করা হয় এবং database পরে asynchronously update করা হয়।

```text
Write
 ↓
Cache
 ↓
DB Later
```

---

# 63. Interview Question — What is Cache Invalidation?

> **Cache invalidation হলো stale বা outdated cache data remove বা invalidate করার process, যাতে system পুরোনো data serve না করে।**

Example:

```text
DB = 550
Cache = 500 ❌

Invalidate Cache
       ↓
Next Request
       ↓
Fresh Data
```

---

# 64. Interview Question — Why not Cache Everything?

কারণ cache-এর:

```text
Memory Limited
+
Data May Change
+
Consistency Required
```

সব data cache করলে:

```text
Memory Cost ↑
Invalidation Complexity ↑
Stale Data Risk ↑
```

তাই সাধারণত high-read, expensive-to-fetch এবং comparatively stable data cache করা হয়।

---

# 65. Interview Question — What happens if Cache fails?

একটি possible architecture:

```text
Application
    ↓
Cache
    ↓
  FAIL
    ↓
Database
```

Application database থেকে data fetch করতে পারে।

কিন্তু তখন:

```text
DB Traffic ↑
```

তাই production system-এ cache failure-এর জন্য fallback, replication, monitoring এবং overload protection দরকার হতে পারে।

---

# 66. Interview Question — What is Redis?

> **Redis হলো একটি in-memory data store যা caching সহ বিভিন্ন low-latency use case-এ ব্যবহৃত হয়।**

Example:

```text
Application
     ↓
Redis
     ↓
Database
```

Redis cache হিসেবে frequently accessed data দ্রুত serve করতে পারে।

---

# 67. Important System Design Questions

Caching design করার সময় নিজেকে জিজ্ঞেস করো:

```text
1. What should I cache?
2. Why should I cache it?
3. Where should the cache live?
4. What should be the cache key?
5. How long should data stay in cache?
6. How do I invalidate stale data?
7. What happens on cache miss?
8. What happens if the cache fails?
9. Which eviction policy should I use?
10. How will I prevent cache stampede?
11. How will I maintain consistency?
12. What is my expected cache hit ratio?
```

---

# 68. Simple Real-Life Mental Model

```text
Database
   ↓
Original Source

Cache
   ↓
Fast Copy

Hit
   ↓
Use Fast Copy

Miss
   ↓
Go to Original Source

Update
   ↓
Keep Cache Consistent
```

---

# 69. One Complete Example

ধরো:

```text
User → Backend → Redis → PostgreSQL
```

User:

```text
GET /user/101
```

### First Request

```text
Redis → MISS ❌
       ↓
PostgreSQL
       ↓
User Data
       ↓
Redis
       ↓
Response
```

### Second Request

```text
Redis → HIT ✅
       ↓
Response
```

### User Update

```text
Age: 25 → 26
```

Possible approaches:

```text
Write-Through:
Cache = 26
DB = 26
```

or:

```text
Cache-Aside:
DB = 26
Cache = Delete
```

or:

```text
Write-Back:
Cache = 26
DB = Later 26
```

---

# 70. Final Mental Model

```text
                    CLIENT
                       │
                       ▼
                   SERVER
                       │
                       ▼
                  ┌─────────┐
                  │  CACHE  │
                  └────┬────┘
                       │
               ┌───────┴───────┐
               │               │
             HIT             MISS
               │               │
               ▼               ▼
          Fast Response      DATABASE
                               │
                               ▼
                           Fresh Data
                               │
                               ▼
                        Store in Cache
                               │
                               ▼
                            Response
```

---

# 🧠 Final Memory Trick

## Caching

```text
REQUEST
   ↓
CACHE
   ↓
HIT  → Fast Response
   ↓
MISS
   ↓
DATABASE
   ↓
STORE IN CACHE
```

## Write-Through

```text
WRITE
  ↓
CACHE + DATABASE
```

## Write-Back

```text
WRITE
  ↓
CACHE
  ↓
DATABASE LATER
```

## Cache-Aside

```text
READ
 ↓
CACHE
 ↓
MISS → DATABASE → CACHE
```

## TTL

```text
CACHE
 ↓
Time Expires
 ↓
Remove Entry
```

## Eviction

```text
Cache Full
   ↓
Remove Some Data
```

## Main Problems

```text
Stale Data
    ↓
Invalidation

Many Misses
    ↓
Database Load

Cache Failure
    ↓
Fallback Required
```

---

# ⭐ সবচেয়ে গুরুত্বপূর্ণ 10টি Point

```text
1. Cache stores frequently accessed data for faster access.

2. Cache reduces latency and database load.

3. Cache can exist at the client, server, or database level.

4. Cache Hit means data is found in cache.

5. Cache Miss means data is not found in cache.

6. Cache is also useful for expensive computations.

7. Cached data can become stale when the original data changes.

8. Write-Through updates cache and database together.

9. Write-Back updates cache first and database asynchronously later.

10. Cache performance comes with consistency and invalidation complexity.
```

---

# 🔥 One-Line Summary

> **Caching = Frequently Used Data → Fast Storage → Faster Response + Lower Database Load**

### Final Formula

```text
REQUEST
   ↓
CACHE
   ↓
HIT → FAST RESPONSE ⚡
   ↓
MISS
   ↓
DATABASE
   ↓
CACHE
   ↓
RESPONSE
```

### Final Concept

```text
Caching = Performance
        +
Consistency Management
```

> **System Design-এ cache শুধু performance বাড়ানোর tool নয়; cache কীভাবে read, write, expire, invalidate এবং fail করবে—সেটাও design-এর গুরুত্বপূর্ণ অংশ।**

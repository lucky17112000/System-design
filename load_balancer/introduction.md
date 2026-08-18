# ⚖️ Load Balancer — System Design Notes

> **Load Balancer** হলো এমন একটি component, যা incoming client request-গুলোকে একাধিক backend server-এর মধ্যে distribute করে যাতে কোনো একটি server অতিরিক্ত overloaded না হয় এবং system-এর **scalability, availability, throughput ও performance** উন্নত হয়।

## 1. Load Balancer কী?

সহজভাবে:

> **Load Balancer = Traffic Cop 🚦**

```mermaid
flowchart LR
    C1[Client 1] --> LB[Load Balancer]
    C2[Client 2] --> LB
    C3[Client 3] --> LB
    LB --> S1[Server 1]
    LB --> S2[Server 2]
    LB --> S3[Server 3]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class C1,C2,C3 client;
    class LB lb;
    class S1,S2,S3 server;
```

---

## 2. কেন Load Balancer দরকার?

একটি মাত্র server থাকলে user বাড়ার সাথে সাথে load বাড়ে।

```mermaid
flowchart TD
    U[Many Clients] --> S[Single Server]
    S --> P[Heavy Load]
    P --> F[Slow / Failure]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef server fill:#F5B7B1,stroke:#C0392B,color:#78281F,stroke-width:3px;
    classDef problem fill:#FAD7A0,stroke:#CA6F1E,color:#784212,stroke-width:2px;
    class U user;
    class S server;
    class P,F problem;
```

---

## 3. Scaling

### Vertical Scaling

একটি server-কে আরও powerful করা।

```text
4 CPU Core + 8 GB RAM
          ↓
32 CPU Core + 128 GB RAM
```

> **Vertical Scaling = Scale Up**

কিন্তু একটি machine-এর hardware limit আছে।

### Horizontal Scaling

আরও server যোগ করা।

```mermaid
flowchart TD
    U[Users] --> S1[Server 1]
    U --> S2[Server 2]
    U --> S3[Server 3]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class U user;
    class S1,S2,S3 server;
```

> **Horizontal Scaling = Scale Out**

---

## 4. Horizontal Scaling-এর সমস্যা

Multiple server যোগ করার পর প্রশ্ন হয়:

> **কোন request কোন server-এ যাবে?**

এখানেই Load Balancer দরকার।

```mermaid
flowchart LR
    U[Users] --> LB[Load Balancer]
    LB --> S1[Server 1]
    LB --> S2[Server 2]
    LB --> S3[Server 3]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class U user;
    class LB lb;
    class S1,S2,S3 server;
```

---

## 5. Load Balancer কীভাবে কাজ করে?

```mermaid
sequenceDiagram
    participant C as Client
    participant LB as Load Balancer
    participant S as Backend Server
    participant DB as Database

    C->>LB: HTTP/HTTPS Request
    LB->>S: Forward Request
    S->>DB: Query
    DB-->>S: Result
    S-->>LB: Response
    LB-->>C: Response
```

সহজ flow:

```text
Client
  ↓
Load Balancer
  ↓
Server
  ↓
Database
  ↓
Server
  ↓
Load Balancer
  ↓
Client
```

---

## 6. Load Balancer-এর Main Goals

- **Even Load Distribution**
- **Higher Throughput**
- **Lower Latency**-তে সাহায্য করা
- **High Availability**
- **Failover**
- **Horizontal Scaling support**

---

# 7. Server Selection Strategy

Load Balancer-এর সবচেয়ে গুরুত্বপূর্ণ কাজ:

> **কোন request কোন server-এ যাবে?**

Common strategies:

1. Random
2. Round Robin
3. Weighted Round Robin
4. Least/Load Based
5. IP Hashing

---

# 8. Random Selection

Randomly server নির্বাচন করা।

```mermaid
flowchart LR
    R1[Request 1] --> LB[Load Balancer]
    R2[Request 2] --> LB
    R3[Request 3] --> LB

    LB -->|Random| S1[Server 1]
    LB -->|Random| S2[Server 2]
    LB -->|Random| S3[Server 3]

    classDef req fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class R1,R2,R3 req;
    class LB lb;
    class S1,S2,S3 server;
```

### সমস্যা

Random distribution uneven হতে পারে।

```text
S1 → 10 requests
S2 → 2 requests
S3 → 1 request
```

---

# 9. Round Robin

Serverগুলোকে loop-এর মতো ব্যবহার করা হয়:

```text
S1 → S2 → S3 → S1 → S2 → S3 → ...
```

```mermaid
flowchart LR
    R1[Request 1] --> S1[Server 1]
    R2[Request 2] --> S2[Server 2]
    R3[Request 3] --> S3[Server 3]
    R4[Request 4] --> S1
    R5[Request 5] --> S2
    R6[Request 6] --> S3

    classDef req fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class R1,R2,R3,R4,R5,R6 req;
    class S1,S2,S3 server;
```

### Example

```text
Request 1 → S1
Request 2 → S2
Request 3 → S3
Request 4 → S1
Request 5 → S2
Request 6 → S3
```

> Simple এবং predictable distribution-এর জন্য useful।

---

# 10. Weighted Round Robin

সব server-এর ক্ষমতা একই না হলে weight দেওয়া যায়।

```text
S1 = Weight 1
S2 = Weight 3
S3 = Weight 1
```

```mermaid
flowchart LR
    LB[Load Balancer]
    LB --> S1[Server 1<br/>Weight 1]
    LB --> S2[Server 2<br/>Weight 3]
    LB --> S2
    LB --> S2
    LB --> S3[Server 3<br/>Weight 1]

    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef normal fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    classDef powerful fill:#D2B4DE,stroke:#8E44AD,color:#512E5F,stroke-width:3px;
    class LB lb;
    class S1,S3 normal;
    class S2 powerful;
```

এখানে `S2` বেশি powerful, তাই বেশি traffic পায়।

---

# 11. Health / Load Based Routing

Load Balancer server-এর current condition দেখে decision নিতে পারে।

```text
S1 → CPU 30%
S2 → CPU 95%
S3 → CPU 40%
```

```mermaid
flowchart TD
    LB[Load Balancer]
    S1[Server 1<br/>CPU 30% ✅]
    S2[Server 2<br/>CPU 95% ⚠️]
    S3[Server 3<br/>CPU 40% ✅]

    LB --> S1
    LB -. Less Traffic .-> S2
    LB --> S3

    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef healthy fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    classDef busy fill:#FAD7A0,stroke:#CA6F1E,color:#784212,stroke-width:3px;
    class LB lb;
    class S1,S3 healthy;
    class S2 busy;
```

---

# 12. Health Check

Load Balancer নিয়মিত server-এর health check করতে পারে।

```mermaid
flowchart TD
    LB[Load Balancer]
    S1[Server 1 ✅]
    S2[Server 2 ❌]
    S3[Server 3 ✅]

    LB -->|Health Check| S1
    LB -->|Health Check| S2
    LB -->|Health Check| S3

    LB -->|Traffic| S1
    LB -. No New Traffic .-> S2
    LB -->|Traffic| S3

    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef healthy fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    classDef failed fill:#F5B7B1,stroke:#C0392B,color:#78281F,stroke-width:3px;
    class LB lb;
    class S1,S3 healthy;
    class S2 failed;
```

যদি কোনো server down হয়, Load Balancer নতুন traffic সেখানে পাঠানো বন্ধ করতে পারে।

---

# 13. IP Hashing

Client-এর IP address hash করে server নির্বাচন করা যায়।

```mermaid
flowchart TD
    C[Client IP]
    H[Hash Function]
    M[Hash % Number of Servers]
    S1[Server 1]
    S2[Server 2]
    S3[Server 3]

    C --> H --> M
    M --> S1
    M --> S2
    M --> S3

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef hash fill:#F5EEF8,stroke:#8E44AD,color:#512E5F,stroke-width:2px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class C client;
    class H,M hash;
    class S1,S2,S3 server;
```

উদাহরণ:

```text
hash(IP) % 3 = 1
```

তাহলে:

```text
Client → Server 1
```

একই mapping rules থাকলে একই client বারবার একই server-এ যেতে পারে।

---

# 14. IP Hashing + Cache

Cache-heavy system-এ এটি useful হতে পারে।

### First Request

```mermaid
flowchart LR
    C[Client 1] --> H[Hash IP]
    H --> S[Server 1]
    S --> M[Cache MISS]
    M --> P[Expensive Computation]
    P --> C2[Store Result in Cache]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef hash fill:#F5EEF8,stroke:#8E44AD,color:#512E5F,stroke-width:2px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    classDef cache fill:#FCF3CF,stroke:#B7950B,color:#7D6608,stroke-width:2px;
    class C client;
    class H hash;
    class S server;
    class M,C2 cache;
```

### Next Request

```mermaid
flowchart LR
    C[Same Client] --> H[Same Hash]
    H --> S[Same Server]
    S --> HIT[Cache HIT ✅]
    HIT --> R[Fast Response]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef hash fill:#F5EEF8,stroke:#8E44AD,color:#512E5F,stroke-width:2px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    classDef hit fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:3px;
    class C client;
    class H hash;
    class S server;
    class HIT,R hit;
```

ফলে:

- Cache reuse হয়
- Unnecessary computation কমে
- Response দ্রুত হতে পারে

---

# 15. Load Balancer এবং Throughput

ধরো একটি server handle করতে পারে:

```text
1000 req/sec
```

তিনটি similar server থাকলে idealized capacity:

```text
1000 + 1000 + 1000
≈ 3000 req/sec
```

```mermaid
flowchart TD
    U[Clients] --> LB[Load Balancer]
    LB --> S1[Server 1<br/>1000 req/sec]
    LB --> S2[Server 2<br/>1000 req/sec]
    LB --> S3[Server 3<br/>1000 req/sec]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class U user;
    class LB lb;
    class S1,S2,S3 server;
```

> বাস্তবে throughput perfectly 3× হয় না; database, network, synchronization এবং অন্যান্য bottleneck থাকতে পারে।

---

# 16. Load Balancer এবং Latency

যদি একটি server overloaded হয়:

```text
Queue ↑
  ↓
Waiting Time ↑
  ↓
Latency ↑
```

Load Balancer অন্য server-এ traffic distribute করতে পারে।

```mermaid
flowchart LR
    R[Incoming Requests] --> LB[Load Balancer]
    LB -. Less Traffic .-> S1[Server 1<br/>Overloaded ⚠️]
    LB --> S2[Server 2<br/>Available ✅]
    LB --> S3[Server 3<br/>Available ✅]

    classDef request fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef overload fill:#FAD7A0,stroke:#CA6F1E,color:#784212,stroke-width:3px;
    classDef healthy fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class R request;
    class LB lb;
    class S1 overload;
    class S2,S3 healthy;
```

> Load Balancer latency কমাতে **সাহায্য করতে পারে**, কিন্তু সব ক্ষেত্রে latency কমবে—এমন guarantee নেই।

---

# 17. Load Balancer শুধু Client → Server-এর মাঝে নয়

## Server → Database

```mermaid
flowchart LR
    A[Application Servers] --> LB[Load Balancer]
    LB --> D1[Database Server 1]
    LB --> D2[Database Server 2]

    classDef app fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef db fill:#F5EEF8,stroke:#8E44AD,color:#512E5F,stroke-width:2px;
    class A app;
    class LB lb;
    class D1,D2 db;
```

## Service → Service

```mermaid
flowchart LR
    A[Service A] --> LB[Load Balancer]
    LB --> B1[Service B - 1]
    LB --> B2[Service B - 2]
    LB --> B3[Service B - 3]

    classDef service fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    class A,B1,B2,B3 service;
    class LB lb;
```

---

# 18. Load Balancer এবং Reverse Proxy

Load Balancer অনেক ক্ষেত্রে Reverse Proxy হিসেবেও কাজ করতে পারে।

```mermaid
flowchart LR
    C[Client]
    P[Load Balancer / Reverse Proxy]
    S1[Backend 1]
    S2[Backend 2]
    S3[Backend 3]

    C --> P
    P --> S1
    P --> S2
    P --> S3

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class C client;
    class P proxy;
    class S1,S2,S3 server;
```

---

# 19. DNS Load Balancing

DNS level-এও traffic distribute করা যেতে পারে।

ধরো:

```text
google.com
```

DNS query-এর response-এ একাধিক IP পাওয়া যেতে পারে।

```mermaid
flowchart TD
    C[Client]
    DNS[DNS Resolver]
    IP1[IP Address 1]
    IP2[IP Address 2]
    IP3[IP Address 3]

    C --> DNS
    DNS --> IP1
    DNS --> IP2
    DNS --> IP3

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef dns fill:#F5EEF8,stroke:#8E44AD,color:#512E5F,stroke-width:3px;
    classDef ip fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class C client;
    class DNS dns;
    class IP1,IP2,IP3 ip;
```

> একটি hostname-এর multiple IP পাওয়া শুধু DNS load balancing-এর কারণেই হবে—এমন নয়। CDN, geographic routing, caching এবং provider infrastructure-এর কারণেও ভিন্ন IP পাওয়া যেতে পারে।

---

# 20. `nslookup` Example

Terminal-এ:

```bash
nslookup google.com
```

এটি DNS query করে IP address দেখতে সাহায্য করে।

Concept:

```text
google.com
    ↓
DNS
    ↓
IP Address
```

একাধিক query-তে ভিন্ন address দেখা যেতে পারে।

---

# 21. Hardware Load Balancer

Dedicated physical device:

```mermaid
flowchart LR
    C[Clients] --> H[Hardware Load Balancer]
    H --> S1[Server 1]
    H --> S2[Server 2]
    H --> S3[Server 3]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef hardware fill:#D7BDE2,stroke:#8E44AD,color:#512E5F,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class C client;
    class H hardware;
    class S1,S2,S3 server;
```

### সুবিধা

- High performance
- Specialized hardware

### অসুবিধা

- Expensive
- Hardware limitation
- Flexibility কম হতে পারে

---

# 22. Software Load Balancer

Software হিসেবে run করে।

Popular examples:

- Nginx
- HAProxy
- Envoy

```mermaid
flowchart LR
    C[Clients] --> S[Software Load Balancer]
    S --> S1[Server 1]
    S --> S2[Server 2]
    S --> S3[Server 3]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef software fill:#FCF3CF,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class C client;
    class S software;
    class S1,S2,S3 server;
```

### সুবিধা

- তুলনামূলকভাবে কম খরচ
- Flexible
- Highly configurable
- Cloud environment-এ জনপ্রিয়

---

# 23. Server Register / Deregister

নতুন server যোগ করলে Load Balancer-কে জানাতে হয়।

### Register

```mermaid
sequenceDiagram
    participant S as New Server
    participant LB as Load Balancer

    S->>LB: Register
    LB-->>S: Accepted
    LB->>S: Start Sending Traffic
```

### Deregister

Server remove করার সময়:

```mermaid
sequenceDiagram
    participant S as Server
    participant LB as Load Balancer

    S->>LB: Deregister / Drain
    LB-->>S: Stop New Traffic
    S-->>LB: Finish Existing Requests
```

---

# 24. Load Balancer + Distributed System

আগে:

```mermaid
flowchart TD
    U[Users] --> S[Single Server]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef server fill:#F5B7B1,stroke:#C0392B,color:#78281F,stroke-width:3px;
    class U user;
    class S server;
```

পরে:

```mermaid
flowchart TD
    U[Users] --> LB[Load Balancer]
    LB --> S1[Server 1]
    LB --> S2[Server 2]
    LB --> S3[Server 3]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class U user;
    class LB lb;
    class S1,S2,S3 server;
```

মনে রাখো:

```text
Horizontal Scaling
        +
Load Balancing
        =
More Capacity
+
Better Availability
```

---

# 25. Main Responsibilities — Quick Table

| Responsibility    | কাজ                                 |
| ----------------- | ----------------------------------- |
| Routing           | Request server-এ পাঠানো             |
| Load Distribution | Traffic ভাগ করা                     |
| Health Check      | Server healthy কিনা দেখা            |
| Failover          | Failed server এড়িয়ে চলা             |
| Scaling Support   | নতুন server ব্যবহার করা             |
| Availability      | Service available রাখতে সাহায্য করা |

---

# 26. Strategies — Quick Table

| Strategy             | কীভাবে কাজ করে                 |
| -------------------- | ------------------------------ |
| Random               | Random server                  |
| Round Robin          | একের পর এক server              |
| Weighted Round Robin | Powerful server-এ বেশি traffic |
| Least / Load Based   | কম loaded server               |
| IP Hashing           | Client IP hash করে server      |

---

# 27. Real-Life Analogy

ধরো একটি hospital:

```mermaid
flowchart LR
    P1[Patient 1]
    P2[Patient 2]
    P3[Patient 3]
    R[Reception / Traffic Controller]
    D1[Doctor 1]
    D2[Doctor 2]
    D3[Doctor 3]

    P1 --> R
    P2 --> R
    P3 --> R
    R --> D1
    R --> D2
    R --> D3

    classDef patient fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef reception fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef doctor fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    class P1,P2,P3 patient;
    class R reception;
    class D1,D2,D3 doctor;
```

এখানে:

```text
Patients  = Clients
Reception = Load Balancer
Doctors   = Backend Servers
```

---

# 28. Exam Definition

### English

> **A load balancer is a system component that distributes incoming client requests across multiple backend servers to improve scalability, availability, and performance.**

### বাংলা

> **Load Balancer হলো এমন একটি component, যা incoming client request-গুলোকে একাধিক backend server-এর মধ্যে distribute করে যাতে system-এর scalability, availability এবং performance উন্নত হয়।**

---

# 29. Interview Answer

> **প্রথমে যদি একটি server সব client request handle করে, user বাড়ার সাথে সাথে server bottleneck হয়ে যেতে পারে। তাই আমরা horizontal scaling করে multiple server যোগ করি। এরপর incoming request-গুলোকে সেই serverগুলোর মধ্যে distribute করার জন্য Load Balancer ব্যবহার করি। Load Balancer client এবং backend server-এর মাঝখানে থাকে এবং round robin, weighted round robin, load-based routing বা hashing-এর মতো strategy ব্যবহার করে server নির্বাচন করে। এটি health check-এর মাধ্যমে unhealthy server এড়িয়ে যেতে পারে এবং system-এর scalability, availability ও throughput improve করতে সাহায্য করে।**

---

# 30. 🧠 20-Second Mental Model

```mermaid
flowchart TD
    U[Many Users]
    LB[Load Balancer]
    S1[Server 1]
    S2[Server 2]
    S3[Server 3]
    DB[(Database)]

    U --> LB
    LB --> S1
    LB --> S2
    LB --> S3
    S1 --> DB
    S2 --> DB
    S3 --> DB

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef lb fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;
    classDef db fill:#F5EEF8,stroke:#8E44AD,color:#512E5F,stroke-width:3px;
    class U user;
    class LB lb;
    class S1,S2,S3 server;
    class DB db;
```

মনে রাখো:

```text
Many Users
    ↓
Load Balancer
    ↓
Multiple Servers
    ↓
Database
```

---

# ⭐ 31. Final Memory Trick

### Load Balancer = Traffic Cop 🚦

> **Client request আসে → Load Balancer সিদ্ধান্ত নেয় → সঠিক server-এ পাঠায়।**

### Horizontal Scaling

> **আরও server যোগ করো।**

### Load Balancer

> **সেই serverগুলোর মধ্যে request ভাগ করো।**

### Main Strategies

```text
Random
Round Robin
Weighted Round Robin
Least / Load Based
IP Hashing
```

### Main Benefits

```text
Scalability
Availability
Throughput ↑
Latency ↓ (often)
Fault Tolerance
```

## 🔑 সবচেয়ে গুরুত্বপূর্ণ লাইন

> **Horizontal Scaling system-এ server-এর সংখ্যা বাড়ায়, আর Load Balancer incoming traffic-কে সেই serverগুলোর মধ্যে intelligently distribute করে যাতে কোনো একটি server bottleneck না হয়ে যায়।**

> **এক লাইনে: Load Balancer = "Which server should handle this request?" এই decision নেওয়ার component।**

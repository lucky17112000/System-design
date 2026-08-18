# 🔄 Forward Proxy vs Reverse Proxy — System Design Notes

## 1. Proxy কী?

**Proxy** হলো একটি intermediary/server, যেটা Client এবং Destination-এর মাঝখানে থেকে request receive করে এবং অন্যদিকে forward করে।

```mermaid
flowchart LR
    C[Client] --> P[Proxy] --> S[Destination Server]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class P proxy;
    class S server;
```

মূল প্রশ্ন:

> **Proxy কার behalf-এ কাজ করছে?**

- **Forward Proxy → Client-এর behalf-এ**
- **Reverse Proxy → Server-এর behalf-এ**

---

# 2. Forward Proxy কী?

> **Forward Proxy Client-এর behalf-এ কাজ করে।**

```mermaid
flowchart LR
    C[Client] -->|Request| FP[Forward Proxy]
    FP -->|Request| S[Internet / Destination Server]
    S -->|Response| FP
    FP -->|Response| C

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#AED6F1,stroke:#2E86C1,color:#154360,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class FP proxy;
    class S server;
```

এখানে Client সাধারণত জানে যে সে Proxy ব্যবহার করছে।

### Basic Flow

```text
Client
  ↓
Forward Proxy
  ↓
Internet / Server
```

---

# 3. Forward Proxy-এর Example: VPN

Direct connection:

```mermaid
flowchart LR
    C[Your Computer] --> S[Website]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class S server;
```

Forward Proxy/VPN ব্যবহার করলে:

```mermaid
flowchart LR
    C[Your Computer] --> P[Forward Proxy / VPN] --> S[Website]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#AED6F1,stroke:#2E86C1,color:#154360,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class P proxy;
    class S server;
```

Website সাধারণত proxy/VPN-এর network address দেখতে পারে, client-এর direct network address-এর পরিবর্তে।

> Forward Proxy client-এর identity/location hide করতে সাহায্য করতে পারে, তবে exact behavior proxy/VPN configuration-এর উপর নির্ভর করে।

---

# 4. Forward Proxy কেন ব্যবহার করা হয়?

Common use case:

### Client Privacy / Network Control

```text
Client
  ↓
Forward Proxy
  ↓
Internet
```

### Corporate Proxy

```mermaid
flowchart LR
    U[Employees] --> P[Company Forward Proxy]
    P --> I[Internet]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#AED6F1,stroke:#2E86C1,color:#154360,stroke-width:3px;
    classDef internet fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class U user;
    class P proxy;
    class I internet;
```

Company Proxy দিয়ে:

- Website filtering
- Access control
- Monitoring
- Traffic policy enforcement

করা যেতে পারে।

### Geographic Access

কিছু proxy/VPN অন্য region-এর network location ব্যবহার করতে পারে, ফলে location-based access policy-তে apparent source region পরিবর্তিত হতে পারে।

---

# 5. Reverse Proxy কী?

> **Reverse Proxy Server-এর behalf-এ কাজ করে।**

```mermaid
flowchart LR
    C[Client] --> RP[Reverse Proxy]
    RP --> S1[Backend Server 1]
    RP --> S2[Backend Server 2]
    RP --> S3[Backend Server 3]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class RP proxy;
    class S1,S2,S3 server;
```

Client সাধারণত backend-এর ভিতরের structure জানে না।

Client শুধু public endpoint-এর সাথে communicate করে।

---

# 6. Reverse Proxy-এর Example

ধরো:

```text
www.example.com
```

Browser request পাঠায়:

```mermaid
flowchart LR
    C[Browser] --> RP[Reverse Proxy]
    RP --> S1[App Server 1]
    RP --> S2[App Server 2]
    RP --> S3[App Server 3]

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class RP proxy;
    class S1,S2,S3 server;
```

Reverse Proxy decide করতে পারে কোন backend server request handle করবে।

Client backend-এর actual server topology জানে না।

---

# 7. Reverse Proxy-এর প্রধান কাজ

## 7.1 Load Balancing

Reverse Proxy backend server-গুলোর মধ্যে request distribute করতে পারে।

```mermaid
flowchart TD
    C[Many Clients]
    RP[Reverse Proxy / Load Balancer]
    S1[Server 1]
    S2[Server 2]
    S3[Server 3]

    C --> RP
    RP --> S1
    RP --> S2
    RP --> S3

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class RP proxy;
    class S1,S2,S3 server;
```

ফল:

- Load distribute করা যায়
- Server overload কমতে পারে
- Availability improve হতে পারে

> **Reverse Proxy এবং Load Balancer একই জিনিস নয়। তবে একটি Reverse Proxy Load Balancing করতে পারে।**

---

## 7.2 Static Asset Caching

Reverse Proxy cache করতে পারে:

- Images
- CSS
- JavaScript
- Fonts
- কিছু static files

```mermaid
flowchart LR
    C[Client] --> RP[Reverse Proxy]
    RP -->|Cache Hit| C
    RP -->|Cache Miss| S[Backend Server]
    S --> RP

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class RP proxy;
    class S server;
```

### Cache Hit

```text
Client
  ↓
Reverse Proxy
  ↓
Cache HIT ✅
  ↓
Response
```

### Cache Miss

```text
Client
  ↓
Reverse Proxy
  ↓
Cache MISS
  ↓
Backend Server
  ↓
Response
```

ফলে:

- Backend load কমতে পারে
- Latency কমতে পারে
- Throughput improve হতে পারে

---

## 7.3 Security

Reverse Proxy backend server-গুলোকে direct public exposure থেকে আড়ালে রাখতে পারে।

```mermaid
flowchart LR
    I[Internet] --> RP[Reverse Proxy]
    RP --> B[Private Backend Servers]

    classDef internet fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef proxy fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef backend fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:3px;

    class I internet;
    class RP proxy;
    class B backend;
```

Reverse Proxy-তে থাকতে পারে:

- TLS termination
- Rate limiting
- Request filtering
- Authentication integration
- WAF integration

---

# 8. Forward Proxy vs Reverse Proxy

| বিষয় | Forward Proxy | Reverse Proxy |
|---|---|---|
| কার behalf-এ কাজ করে? | Client | Server |
| অবস্থান | Client-side | Server-এর সামনে |
| Basic Flow | Client → Proxy → Internet | Client → Proxy → Backend |
| Client কি Proxy জানে? | সাধারণত হ্যাঁ | Backend structure সম্পর্কে সাধারণত না |
| Main goal | Client traffic control/privacy | Backend protection/routing |
| Common use case | Corporate Proxy, VPN | Load Balancing, Caching |
| Backend infrastructure hide করে? | Main purpose নয় | হ্যাঁ, করতে পারে |

### সবচেয়ে সহজে

```text
Forward Proxy → Client-এর জন্য
Reverse Proxy → Server-এর জন্য
```

---

# 9. Restaurant Analogy

## Forward Proxy

তুমি অন্য কাউকে দিয়ে restaurant-এ order পাঠালে:

```text
You
 ↓
Messenger
 ↓
Restaurant
```

Messenger তোমার behalf-এ কাজ করছে।

```text
Messenger = Forward Proxy
```

## Reverse Proxy

Restaurant-এর সামনে Receptionist:

```text
You
 ↓
Receptionist
 ↓
Chef 1 / Chef 2 / Chef 3
```

তুমি জানো না কোন chef তোমার food তৈরি করছে।

```text
Receptionist = Reverse Proxy
```

---

# 10. Forward + Reverse Proxy একসাথে

একটি বড় system-এ দুটোই থাকতে পারে:

```mermaid
flowchart LR
    C[Client]
    FP[Forward Proxy]
    I[Internet]
    RP[Reverse Proxy]
    S1[Backend 1]
    S2[Backend 2]
    S3[Backend 3]

    C --> FP
    FP --> I
    I --> RP

    RP --> S1
    RP --> S2
    RP --> S3

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef forward fill:#AED6F1,stroke:#2E86C1,color:#154360,stroke-width:3px;
    classDef internet fill:#E8F8F5,stroke:#17A589,color:#0E6251,stroke-width:2px;
    classDef reverse fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class FP forward;
    class I internet;
    class RP reverse;
    class S1,S2,S3 server;
```

পুরো flow:

```text
Client
  ↓
Forward Proxy
  ↓
Internet
  ↓
Reverse Proxy
  ↓
Backend Servers
```

---

# 11. Reverse Proxy vs Load Balancer

এটা খুব important:

```text
Reverse Proxy
     │
     ├── Routing
     ├── Caching
     ├── Security
     ├── TLS Termination
     └── Load Balancing
```

অর্থাৎ:

> **Reverse Proxy একটি broader role হতে পারে, আর Load Balancing তার একটি function।**

Nginx, HAProxy, Envoy-এর মতো software reverse proxy এবং load balancing—দুটোই করতে পারে।

---

# 12. Reverse Proxy এবং CDN

CDN edge architecture অনেক ক্ষেত্রে reverse-proxy-like pattern ব্যবহার করে।

```mermaid
flowchart LR
    U[User] --> CDN[CDN Edge]
    CDN --> O[Origin Server]

    classDef user fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef cdn fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef origin fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class U user;
    class CDN cdn;
    class O origin;
```

Cache-এ content থাকলে:

```text
User
 ↓
CDN
 ↓
Cache HIT ✅
 ↓
Response
```

Origin server-এ request নাও যেতে পারে।

---

# 13. Interview Answer

### English

> **A forward proxy acts on behalf of the client, while a reverse proxy acts on behalf of the server. A forward proxy is commonly used for client-side privacy, access control, filtering, or routing outbound traffic. A reverse proxy sits in front of backend servers and can hide the backend infrastructure while providing features such as load balancing, caching, TLS termination, and security controls.**

### বাংলা

> **Forward Proxy client-এর behalf-এ কাজ করে, আর Reverse Proxy server-এর behalf-এ কাজ করে। Forward Proxy client-side traffic control, privacy, filtering বা outbound routing-এর জন্য ব্যবহার করা হয়। Reverse Proxy backend server-এর সামনে থাকে এবং backend infrastructure hide করার পাশাপাশি load balancing, caching, TLS termination এবং security-এর মতো কাজ করতে পারে।**

---

# 14. 🧠 10-Second Memory Trick

```text
FORWARD PROXY
      ↓
Client-এর হয়ে বাইরে কথা বলে

REVERSE PROXY
      ↓
Server-এর হয়ে Client-এর সাথে কথা বলে
```

আরও সহজ:

```text
Forward → Client Side
Reverse → Server Side
```

---

# 15. Final Mental Model

```mermaid
flowchart LR
    C[Client]
    FP[Forward Proxy]
    Internet[Internet]
    RP[Reverse Proxy]
    S1[Server 1]
    S2[Server 2]
    S3[Server 3]

    C --> FP
    FP --> Internet
    Internet --> RP
    RP --> S1
    RP --> S2
    RP --> S3

    classDef client fill:#D6EAF8,stroke:#2874A6,color:#154360,stroke-width:2px;
    classDef forward fill:#AED6F1,stroke:#2E86C1,color:#154360,stroke-width:3px;
    classDef internet fill:#E8F8F5,stroke:#17A589,color:#0E6251,stroke-width:2px;
    classDef reverse fill:#F9E79F,stroke:#B7950B,color:#7D6608,stroke-width:3px;
    classDef server fill:#D5F5E3,stroke:#229954,color:#145A32,stroke-width:2px;

    class C client;
    class FP forward;
    class Internet internet;
    class RP reverse;
    class S1,S2,S3 server;
```

## 🔑 Final Rule

> **Forward Proxy → Client-এর behalf-এ।**

> **Reverse Proxy → Server-এর behalf-এ।**

> **Reverse Proxy load balancing, caching, security এবং routing করতে পারে।**

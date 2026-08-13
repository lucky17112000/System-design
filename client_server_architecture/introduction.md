# System Design Notes: Client--Server Architecture

## ১. Client--Server Architecture কী?

**Client--Server Architecture** হলো এমন একটি system architecture যেখানে
**Client কোনো service বা data-এর জন্য Server-এর কাছে request পাঠায়**, এবং
Server সেই request process করে Client-কে response দেয়।

সহজভাবে:

> **Client = যে service চায়**\
> **Server = যে service দেয়**

### Basic Flow

```text
Client  →  Request  →  Server
Client  ←  Response ←  Server
```

---

# ২. Client কী?

**Client** হলো এমন একটি device বা application, যেটি Server-এর কাছে
request পাঠায়।

উদাহরণ:

- Web Browser --- Chrome, Firefox
- Mobile Application
- Desktop Application
- Frontend Application

যেমন তুমি Chrome-এ লিখলে:

```text
www.amazon.com
```

এখানে **Chrome Browser হলো Client**।

Client Server-এর কাছে request পাঠায়:

```text
Client → Request → Server
```

---

# ৩. Server কী?

**Server** হলো এমন একটি computer বা software system, যেটি Client-এর
request গ্রহণ করে, process করে এবং response পাঠায়।

Server বিভিন্ন কাজ করতে পারে:

- Business Logic execute করা
- Database থেকে data নেওয়া
- User Authentication করা
- User Authorization করা
- File provide করা
- API provide করা
- Payment process করা
- অন্য service-এর সাথে communicate করা

### উদাহরণ

```text
Client:
"আমার profile information দাও।"

        ↓

Server:
User-এর পরিচয় check করবে এবং data চাইবে।

        ↓

Database:
User-এর information return করবে।

        ↓

Server:
Information Client-এর কাছে পাঠাবে।
```

---

# ৪. Basic Client--Server Architecture

সবচেয়ে basic architecture:

```text
+----------+              +----------+
|          |              |          |
|  Client  | -----------> |  Server  |
|          |   Request    |          |
+----------+              +----------+
                               |
                               |
                               ▼
                         +-----------+
                         | Database  |
                         +-----------+
```

সাধারণ flow:

```text
Client → Server → Database
```

তারপর response ফিরে আসে:

```text
Database → Server → Client
```

---

# ৫. Request এবং Response

Client--Server Architecture-এর সবচেয়ে গুরুত্বপূর্ণ concept হলো **Request এবং
Response**।

## Request কী?

Client যখন Server-এর কাছে কোনো service বা data চায়, সেটাকে **Request**
বলে।

উদাহরণ:

```http
GET /users/101
```

এর অর্থ:

> "User ID 101-এর information আমাকে দাও।"

## Response কী?

Server request process করার পরে Client-কে যে result পাঠায়, সেটাকে
**Response** বলে।

উদাহরণ:

```json
{
  "id": 101,
  "name": "Rahim",
  "email": "rahim@example.com"
}
```

### পুরো Flow

```text
Client
   │
   │ Request
   ▼
Server
   │
   │ Process
   ▼
Database
   │
   │ Data
   ▼
Server
   │
   │ Response
   ▼
Client
```

---

# ৬. Real-Life Example: E-Commerce Website

ধরো তুমি একটি E-Commerce website ব্যবহার করছো এবং **"My Orders"** button-এ
click করলে।

## Step ১: Client Request পাঠাবে

```text
Browser
   ↓
GET /orders
```

## Step ২: Server Request গ্রহণ করবে

Server check করবে:

```text
এই user কে?
```

তারপর Authentication এবং Authorization করবে।

## Step ৩: Server Database-এ Query করবে

উদাহরণ:

```sql
SELECT * FROM orders
WHERE user_id = 101;
```

## Step ৪: Database Data Return করবে

```text
Order 1
Order 2
Order 3
```

## Step ৫: Server Response তৈরি করবে

```json
{
  "orders": [
    { "id": 1, "amount": 500 },
    { "id": 2, "amount": 800 }
  ]
}
```

## Step ৬: Client Result দেখাবে

Browser-এ দেখা যেতে পারে:

```text
My Orders

Order #1    ৳500
Order #2    ৳800
```

---

# ৭. Client কি সরাসরি Database-এর সাথে কথা বলে?

সাধারণত **না**।

ভালো এবং নিরাপদ architecture:

```text
Client
   ↓
Backend Server
   ↓
Database
```

সাধারণত এমন করা হয় না:

```text
Client
   ↓
Database
```

কারণ Server একটি **middle layer** হিসেবে কাজ করে।

Server-এর মাধ্যমে:

- Authentication করা যায়
- Authorization করা যায়
- Business Logic execute করা যায়
- Input validation করা যায়
- Database access control করা যায়
- Security বাড়ানো যায়

### Banking Example

ধরো তুমি একটি Banking Application ব্যবহার করছো।

```text
Client
   ↓
"আমার account information দেখাও"
   ↓
Server
   ↓
Authentication
   ↓
Authorization
   ↓
Database
```

Server check করবে:

> এই user কি সত্যিই এই information দেখার permission রাখে?

---

# ৮. বড় System-এ Client--Server Architecture

System বড় হলে আরও অনেক component যোগ করা হয়।

```text
                    Internet
                       │
                       ▼
                 +-----------+
                 |  Client   |
                 | Browser   |
                 | Mobile App|
                 +-----------+
                       │
                       ▼
                +-------------+
                | Load        |
                | Balancer    |
                +-------------+
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Server 1  Server 2  Server 3
             │         │         │
             └─────────┼─────────┘
                       ▼
                    Database
```

আরও বড় system হতে পারে:

```text
Client
   ↓
DNS
   ↓
Load Balancer
   ↓
API Server
   ↓
Cache
   ↓
Database
   ↓
Message Queue
   ↓
Other Services
```

---

# ৯. Client--Server Architecture-এর সুবিধা

## ৯.১ Centralized Data

Data একটি central Server/Database-এ রাখা যায়।

```text
Client 1 ──┐
Client 2 ──┼──> Server ──> Database
Client 3 ──┘
```

এতে data management সহজ হয়।

---

## ৯.২ Security

Server Database এবং sensitive resource-এর access control করতে পারে।

---

## ৯.৩ Easy Maintenance

Server-side code update করলে সব Client-এর জন্য পরিবর্তন apply করা যায়।

---

## ৯.৪ Scalability

Traffic বাড়লে আরও Server যোগ করা যায়।

```text
1 Server
   ↓
3 Servers
   ↓
10 Servers
   ↓
100 Servers
```

---

## ৯.৫ Multiple Clients

একটি Server অনেক ধরনের Client-কে service দিতে পারে।

```text
             ┌── Web Browser
             │
             ├── Mobile App
Server ──────┤
             ├── Desktop App
             │
             └── Other Services
```

---

# ১০. Client--Server Architecture-এর অসুবিধা

## ১০.১ Server Down হলে

যদি শুধু একটি Server থাকে এবং সেটি down হয়ে যায়:

```text
Client
   ↓
Server ❌
```

তাহলে service unavailable হয়ে যেতে পারে।

বড় system-এ এজন্য multiple Server এবং redundancy ব্যবহার করা হয়।

---

## ১০.২ Network Dependency

Client এবং Server-এর মধ্যে network connection প্রয়োজন।

```text
Client ──X── Server
```

Network সমস্যা হলে communication বন্ধ হয়ে যেতে পারে।

---

## ১০.৩ Server Load

অনেক Client একই সময়ে request করলে Server-এর উপর অনেক load পড়তে পারে।

```text
1,000,000 Clients
       ↓
    Server
       ↓
   Heavy Load
```

এজন্য বড় system-এ ব্যবহার করা হয়:

- Load Balancing
- Caching
- Horizontal Scaling
- Database Scaling
- Message Queue

---

# ১১. Client--Server বনাম Peer-to-Peer

এই দুই architecture-কে গুলিয়ে ফেলবে না।

## Client--Server

```text
Client → Server
Client → Server
Client → Server
```

এখানে Server-এর একটি central role থাকে।

## Peer-to-Peer

```text
Computer A ↔ Computer B
     ↕           ↕
Computer C ↔ Computer D
```

এখানে একটি device একইসাথে Client এবং Server-এর role পালন করতে পারে।

---

# ১২. System Design-এ Client--Server Architecture কেন গুরুত্বপূর্ণ?

**Client--Server Architecture হলো System Design-এর একটি fundamental
concept।**

যখন আমরা নিচের মতো system design করি:

- Facebook
- YouTube
- Netflix
- Uber
- Food Delivery System
- Banking System
- E-Commerce System

তখন basic idea সাধারণত:

```text
Client
   ↓
Server
   ↓
Database
```

System বড় হলে এর সাথে যুক্ত হতে পারে:

```text
Client
   ↓
DNS
   ↓
Load Balancer
   ↓
API Server
   ↓
Cache
   ↓
Database
   ↓
Message Queue
   ↓
Other Services
```

তাই:

> **Client--Server Architecture হলো বড় System Design শেখার একটি
> foundation।**

---

# ১৩. Chrome-এ `amazon.com` লিখলে কী হয়?

এখন একটি real-world example দিয়ে পুরো বিষয়টি বুঝি।

ধরো তুমি Chrome খুলে লিখলে:

```text
amazon.com
```

এখন Browser-এর একটি সমস্যা আছে:

> **"amazon.com কোন IP Address-এর Server-এ আছে?"**

কারণ Internet-এর মাধ্যমে network communication করার জন্য IP Address
প্রয়োজন।

---

# ১৪. Step ১: Browser/OS DNS Information খোঁজে

Browser বা Operating System প্রথমে check করতে পারে যে `amazon.com`-এর IP
Address আগে থেকেই cache-এ আছে কি না।

উদাহরণ:

```text
amazon.com → 18.xxx.xxx.xxx
```

যদি valid cache information থাকে, তাহলে নতুন DNS lookup-এর প্রয়োজন নাও হতে
পারে।

যদি না থাকে, তাহলে DNS resolution শুরু হয়।

---

# ১৫. Step ২: DNS Query

তখন Computer DNS Resolver-কে জিজ্ঞাসা করে:

```text
"amazon.com-এর IP Address কী?"
```

Conceptually:

```text
Your Computer
     │
     │ DNS Query
     │ "IP of amazon.com?"
     ▼
DNS Resolver
```

DNS-এর প্রধান কাজ:

```text
Domain Name → IP Address
```

যেমন:

```text
amazon.com
     ↓
18.xxx.xxx.xxx
```

বড় service যেমন Amazon-এর একাধিক IP Address থাকতে পারে। DNS, CDN এবং
load-balancing system-এর কারণে বিভিন্ন user ভিন্ন IP address পেতে পারে।

---

# ১৬. Step ৩: DNS IP Address Return করে

DNS Resolver একটি IP Address return করে।

উদাহরণ:

```text
amazon.com
     ↓
18.xxx.xxx.xxx
```

এখন Computer জানে কোন destination IP-তে যেতে হবে।

Flow:

```text
Chrome
  │
  │ DNS Query
  ▼
DNS Resolver
  │
  │ IP Address
  ▼
Chrome / OS
```

---

# ১৭. Step ৪: Browser Server-এর সাথে Connection তৈরি করে

শুধু IP Address জানলেই HTTP request পাঠানো শেষ নয়।

সাধারণ HTTPS website-এর ক্ষেত্রে Browser প্রথমে network connection তৈরি করে
এবং তারপর TLS handshake-এর মাধ্যমে secure encrypted connection তৈরি করে।

সহজভাবে:

```text
Browser
   │
   │ Connect to Server IP
   ▼
Amazon Server
```

HTTPS-এর ক্ষেত্রে:

```text
Browser
   │
   ├── TCP Connection
   │
   ├── TLS Handshake
   │
   ▼
Secure HTTPS Connection
```

---

# ১৮. Step ৫: Browser HTTP Request পাঠায়

Connection তৈরি হওয়ার পরে Browser Server-এর কাছে HTTP Request পাঠায়।

একটি simplified request:

```http
GET / HTTP/1.1
Host: amazon.com
```

এর অর্থ:

> **"amazon.com-এর homepage আমাকে দাও।"**

`Host` header Server-কে জানাতে সাহায্য করে কোন hostname-এর জন্য request
করা হয়েছে।

---

# ১৯. HTTP Request-এর মধ্যে কি আমার IP Address থাকে?

এটা খুব গুরুত্বপূর্ণ।

অনেকে বলে:

> "HTTP Request-এর মধ্যে আমার IP Address থাকে।"

এভাবে বলা **technically accurate নয়**।

বরং সঠিকভাবে বলা উচিত:

> **Client-এর Source IP Address IP Packet-এর Network Layer-এর অংশ হিসেবে
> থাকে।**

অন্যদিকে HTTP Request-এর মধ্যে থাকে Application-level information যেমন:

- HTTP Method
- Path
- Host Header
- Other Headers
- Cookies
- Request Body

Conceptually:

```text
IP Packet
├── Source IP
├── Destination IP
│
└── HTTP/HTTPS Data
    ├── Method
    ├── Path
    ├── Host
    ├── Headers
    └── Body
```

এই distinction Networking এবং System Design-এ খুব গুরুত্বপূর্ণ।

---

# ২০. Source IP এবং Destination IP

ধরো উদাহরণ হিসেবে তোমার public IP:

```text
203.0.113.25
```

এবং Amazon-এর destination IP:

```text
18.xxx.xxx.xxx
```

তাহলে IP Packet conceptually এমন হতে পারে:

```text
+--------------------------------+
| Source IP      | 203.0.113.25  |
| Destination IP | 18.xxx.xxx.xxx|
+--------------------------------+
| TCP Information                |
+--------------------------------+
| HTTPS/HTTP Data                |
+--------------------------------+
```

## Source IP

```text
203.0.113.25
```

এর অর্থ:

> Packet কোথা থেকে এসেছে?

## Destination IP

```text
18.xxx.xxx.xxx
```

এর অর্থ:

> Packet কোথায় যাবে?

---

# ২১. Server কীভাবে Response সঠিক জায়গায় পাঠায়?

এটাই সবচেয়ে গুরুত্বপূর্ণ বিষয়।

Network packet-এর addressing information ব্যবহার করে traffic route করা
হয়।

Request-এর simplified flow:

```text
Your Computer
Source IP = Your Public IP
       │
       ▼
Internet
       │
       ▼
Amazon
Destination IP = Amazon IP
```

Response-এর flow বিপরীত দিকে:

```text
Amazon
Source IP = Amazon IP
       │
       ▼
Internet
       │
       ▼
Your Computer
Destination IP = Your Public IP
```

অর্থাৎ response packet-এর ক্ষেত্রে:

```text
Source IP      = Amazon IP
Destination IP = Your Public IP
```

---

# ২২. Private IP এবং Public IP

এটি খুব গুরুত্বপূর্ণ।

ধরো তোমার Computer-এর private IP:

```text
192.168.1.10
```

এবং তোমার Router-এর public IP:

```text
103.xxx.xxx.xxx
```

তাহলে তোমার Computer সাধারণত NAT (Network Address Translation)-এর মাধ্যমে
Internet-এ communicate করতে পারে।

Flow:

```text
Your PC
Private IP:
192.168.1.10
      │
      ▼
   Router
     NAT
      │
      ▼
Public IP:
103.xxx.xxx.xxx
      │
      ▼
Internet
      │
      ▼
Amazon
```

Internet-এর দিকে যাওয়ার সময় তোমার private IP সাধারণত Internet-এর public
destination হিসেবে ব্যবহার করা যায় না।

Router/NAT connection-এর mapping ধরে রাখে এবং response এলে সেটি সঠিক
internal device-এর কাছে forward করে।

---

# ২৩. পুরো `amazon.com` Flow

এখন সবকিছু একসাথে দেখি:

```text
User types:
amazon.com
       │
       ▼
Browser / OS checks cache
       │
       ▼
DNS Resolution
       │
       ▼
Amazon IP Address
       │
       ▼
TCP Connection
       │
       ▼
TLS Handshake (HTTPS)
       │
       ▼
HTTP Request
       │
       ▼
Internet Routing
       │
       ▼
Amazon Server
       │
       ▼
HTTP Response
       │
       ▼
Your Public IP
       │
       ▼
NAT / Router
       │
       ▼
Your Browser
```

---

# ২৪. HTTP এবং IP Packet কীভাবে একসাথে কাজ করে?

এখানে বিভিন্ন Networking Layer একসাথে কাজ করে।

```text
Application Layer
        ↓
HTTP / HTTPS

Transport Layer
        ↓
TCP

Network Layer
        ↓
IP

Data Link Layer
        ↓
Ethernet / Wi-Fi
```

সহজভাবে:

- **HTTP/HTTPS** → Client কী চায় সেটা বোঝায়
- **TCP** → Data transport এবং reliable connection-এর জন্য কাজ করে
- **IP** → Packet কোথা থেকে এসেছে এবং কোথায় যাবে সেটা নির্দেশ করে
- **Ethernet/Wi-Fi** → Local network-এর মধ্যে data পাঠাতে সাহায্য করে

---

# ২৫. Real-Life Analogy

ধরো তুমি Amazon-কে একটি চিঠি পাঠাচ্ছো।

Envelope-এর মধ্যে:

```text
FROM:
তোমার Address

TO:
Amazon-এর Address

CONTENT:
"আমি Homepage চাই।"
```

Amazon যখন reply করবে:

```text
FROM:
Amazon-এর Address

TO:
তোমার Address

CONTENT:
"এই নাও Homepage।"
```

Internet-এ IP Address এই address-এর মতো গুরুত্বপূর্ণ ভূমিকা পালন করে।

---

# ২৬. একটি গুরুত্বপূর্ণ Technical Correction

Beginner-level explanation-এ অনেকে বলে:

> "HTTP Request-এর মধ্যে আমার IP Address থাকে, তাই Server জানে কোথায়
> Response পাঠাতে হবে।"

আরও technically accurate explanation হলো:

> **Client-এর Source IP Address IP Packet-এর মধ্যে থাকে, আর HTTP
> Request-এর মধ্যে Application-level information যেমন HTTP Method, Path,
> Host Header, Cookies এবং Body থাকে। Server response network-এর মাধ্যমে
> client-এর দিকে পাঠায় এবং connection ও network addressing information
> ব্যবহার করে response সঠিক জায়গায় পৌঁছে। NAT থাকলে Router/NAT device
> Internet-facing connection-কে সঠিক internal device-এর সাথে map করে।**

---

# ২৭. Exam-এর জন্য Answer

Exam-এ Client--Server এবং `amazon.com` flow explain করতে চাইলে এভাবে
লিখতে পারো:

> যখন একজন user Browser-এ `amazon.com` লিখে, Browser প্রথমে DNS-এর মাধ্যমে
> domain name-এর corresponding IP Address খুঁজে বের করে। IP Address পাওয়ার
> পরে Browser destination Server-এর সাথে network connection establish
> করে এবং HTTPS হলে TLS handshake সম্পন্ন করে। এরপর Browser Server-এর কাছে
> HTTP Request পাঠায়। এই request IP Packet-এর মাধ্যমে Internet-এর মধ্য
> দিয়ে Server-এর কাছে পৌঁছায়। IP Packet-এ Source এবং Destination IP
> Address থাকে, তাই network devices packet-কে সঠিক destination-এর দিকে
> route করতে পারে। Server request process করে HTTP Response তৈরি করে এবং
> response Client-এর দিকে পাঠায়। Client যদি NAT-এর পিছনে থাকে, তাহলে
> Router/NAT response-টি সঠিক internal device-এর কাছে forward করে।

---

# ২৮. সহজে মনে রাখার Trick

```text
DNS = Domain Name → IP Address

IP = Packet কোথায় যাবে

HTTP = Client কী চায়

TCP = Data Transport / Reliable Connection

TLS = HTTPS-এর Encryption / Security

NAT = Private Network ↔ Public Network Mapping

Server = Request Process করে

Response = Server-এর Result
```

## Final Mental Model

```text
             DOMAIN NAME
                  │
                  ▼
                DNS
                  │
             IP Address
                  │
                  ▼
              Connection
                  │
             HTTPS Request
                  │
                  ▼
               Internet
                  │
                  ▼
              Web Server
                  │
              Processing
                  │
                  ▼
             HTTP Response
                  │
                  ▼
             Your Network
                  │
                NAT
                  │
                  ▼
              Your Browser
```

---

# ২৯. সবচেয়ে গুরুত্বপূর্ণ বিষয়

এই পুরো বিষয়টি এক লাইনে মনে রাখো:

> **DNS নামকে IP Address-এ convert করে → IP packet-কে destination-এর
> দিকে route করতে সাহায্য করে → HTTP বলে Client কী চায় → Server request
> process করে → Response network-এর মাধ্যমে Client-এর কাছে ফিরে আসে → NAT
> থাকলে Router response-কে সঠিক internal device-এর কাছে পৌঁছে দেয়।**

এই flow বুঝে গেলে পরবর্তী System Design topics যেমন:

- Load Balancer
- Reverse Proxy
- API Gateway
- Caching
- Database
- CDN
- Message Queue
- Microservices
- Database Sharding

বোঝা অনেক সহজ হবে।

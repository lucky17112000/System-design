# Chapter: Distributed Systems

## 1. Distributed System কী?

**Distributed System** হলো একাধিক independent computer বা process-এর একটি সমষ্টি, যারা **network-এর মাধ্যমে একে অপরের সাথে communicate ও coordinate করে** এবং একটি **common goal** অর্জনের জন্য একসাথে কাজ করে।

```text
Multiple Machines
        +
Network
        +
Multiple Processes / Components
        +
Communication & Coordination
        +
Common Goal
        =
Distributed System
```

> শুধু অনেকগুলো computer থাকলেই Distributed System হয় না। তাদের network-এর মাধ্যমে communication এবং coordination করে একসাথে কাজ করতে হবে।

---

# 2. User কেন Distributed System দেখতে পায় না?

তুমি যখন online shopping, banking, food delivery বা অন্য কোনো online service ব্যবহার করো, তখন তোমার কাছে মনে হয়:

> "আমি শুধু একটি Website-এর সাথে interact করছি।"

কিন্তু Website-এর আড়ালে অনেকগুলো component বা machine কাজ করতে পারে।

```text
                    User
                      │
                      ▼
                  Website
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      User Service  Product     Payment
                    Service      Service
          │           │           │
          ▼           ▼           ▼
        Database    Database   Payment System
```

তুমি এই সব system আলাদাভাবে দেখছো না। তুমি শুধু একটি **unified service** দেখতে পাচ্ছো।

---

# 3. Centralized System কী?

Distributed System বোঝার আগে **Centralized System** বোঝা দরকার।

একটি Centralized System-এ মূল কাজ একটি machine বা একটি central system-এর উপর নির্ভর করে।

```text
             Users
           /   |             /    |             ▼     ▼     ▼
            Server
         (One Machine)
```

উদাহরণ:

তুমি একটি Bitcoin trading service তৈরি করলে এবং পুরো application একটি computer-এ host করলে:

```text
Users
  ↓
Your Computer
  ↓
Bitcoin Trading Application
```

এটি একটি simple centralized architecture।

---

# 4. Centralized System-এর সমস্যা

ধরো শুরুতে তোমার application-এ মাত্র ১০ জন user।

সব ঠিক আছে।

তারপর user বাড়তে লাগল:

```text
10
 ↓
100
 ↓
10,000
 ↓
1,000,000
```

এখন একটি machine-এর উপর অনেক load পড়বে।

```text
Millions of Requests
        ↓
     1 Server
        ↓
    Heavy Load
```

সমস্যা হতে পারে:

- CPU বেশি ব্যবহার
- Memory বেশি ব্যবহার
- Server overload
- Service downtime
- High latency
- Hardware failure-এর ঝুঁকি

---

# 5. Vertical Scaling কী?

Server-এর load বেড়ে গেলে প্রথমে তুমি একই machine-টিকে আরও powerful করতে পারো।

```text
Before

CPU = 4 Core
RAM = 8 GB
```

তারপর:

```text
After

CPU = 32 Core
RAM = 128 GB
```

এটাকে বলে:

> **Vertical Scaling (Scale Up)**

অর্থাৎ:

> **একটি machine-কে আরও powerful করা।**

```text
        One Server
            │
     Upgrade CPU/RAM
            │
            ▼
    More Powerful Server
```

---

# 6. Vertical Scaling-এর Limitations

## 6.1 Hardware Limit

একটি machine-এ unlimited CPU বা RAM যোগ করা যায় না।

```text
8 GB
 ↓
16 GB
 ↓
32 GB
 ↓
64 GB
 ↓
128 GB
 ↓
Hardware Limit
```

---

## 6.2 Single Point of Failure

যদি পুরো application একটি machine-এ চলে:

```text
Users
  ↓
One Server
  ↓
Server ❌
```

তাহলে পুরো service বন্ধ হয়ে যেতে পারে।

এটাকে বলে:

> **Single Point of Failure (SPOF)**

অর্থাৎ একটি component fail করলে পুরো system fail করতে পারে।

---

## 6.3 Maintenance সমস্যা

একটি মাত্র server থাকলে maintenance বা restart-এর সময় পুরো service unavailable হতে পারে।

---

## 6.4 Distance এবং Latency

ধরো server বাংলাদেশে, কিন্তু user Japan থেকে service ব্যবহার করছে।

```text
Japan User
     ↓
Long Network Distance
     ↓
Bangladesh Server
```

Distance এবং network conditions-এর কারণে latency বাড়তে পারে।

---

# 7. Horizontal Scaling কী?

একটি machine আরও powerful করার পরিবর্তে আরও machine/server যোগ করা যায়।

এটাকে বলে:

> **Horizontal Scaling (Scale Out)**

উদাহরণ:

```text
1 Server
   ↓
3 Servers
   ↓
10 Servers
   ↓
100 Servers
```

```text
             Users
               │
               ▼
          Load Balancer
        /       |              ▼        ▼        ▼
     Server 1 Server 2 Server 3
```

Load Balancer বিভিন্ন request বিভিন্ন server-এ পাঠাতে পারে।

---

# 8. Vertical Scaling বনাম Horizontal Scaling

| Vertical Scaling                      | Horizontal Scaling               |
| ------------------------------------- | -------------------------------- |
| একটি server powerful করা              | নতুন server যোগ করা              |
| CPU/RAM বাড়ানো                        | Machine সংখ্যা বাড়ানো            |
| Scale Up                              | Scale Out                        |
| Hardware limit গুরুত্বপূর্ণ           | Multiple machine ব্যবহার করা যায় |
| Single machine-এর উপর বেশি dependence | Multiple machine                 |
| তুলনামূলকভাবে সহজ                     | Architecture বেশি complex        |

সহজে মনে রাখো:

> **Vertical = বড় machine**  
> **Horizontal = বেশি machine**

---

# 9. Distributed System কেন দরকার?

Application বড় হলে একটি machine-এর উপর নির্ভর করা সমস্যা তৈরি করতে পারে।

Distributed System ব্যবহার করে application-এর কাজ একাধিক machine বা component-এর মধ্যে distribute করা যায়।

```text
             Users
               │
               ▼
          Load Balancer
        /       |              ▼        ▼        ▼
     Server 1 Server 2 Server 3
```

এখন সব load একটি machine-এর উপর নেই।

---

# 10. Distributed System এবং Fault Tolerance

ধরো:

```text
Server 1 ✅
Server 2 ❌
Server 3 ✅
```

যদি system redundancy সহ design করা থাকে, তাহলে Server 2 fail করলেও অন্য serverগুলো service চালিয়ে যেতে পারে।

```text
           Load Balancer
          /      |               ▼       ▼       ▼
       S1 ✅    S2 ❌    S3 ✅
```

এর লক্ষ্য হতে পারে:

> **একটি machine-এর failure যেন পুরো service বন্ধ না করে।**

এটি **Fault Tolerance** এবং **High Availability** design-এর সাথে সম্পর্কিত।

---

# 11. Latency কী?

**Latency** হলো request এবং response-এর মধ্যে delay।

```text
User → Server
```

Server অনেক দূরে হলে বা network congestion থাকলে latency বাড়তে পারে।

Distributed architecture-এ user-এর কাছাকাছি region-এ service/server রাখা যেতে পারে।

```text
Bangladesh User → Asia Server
Europe User → Europe Server
USA User → USA Server
```

তবে:

> Distributed System থাকলেই latency automatically কমে না। Server placement, network এবং architecture সঠিকভাবে design করতে হয়।

---

# 12. Distributed System-এর মূল বৈশিষ্ট্য

একটি Distributed System-এ সাধারণত থাকে:

### Multiple Machines / Nodes

```text
Machine 1
Machine 2
Machine 3
```

### Network Communication

```text
Machine 1
    ↕
Network
    ↕
Machine 2
    ↕
Network
    ↕
Machine 3
```

### Coordination

Machine/processগুলো একে অপরের সাথে coordinate করে।

### Common Goal

সবাই মিলে একটি shared কাজ বা service সম্পন্ন করে।

---

# 13. Distributed System-এর উদাহরণ

একটি online shopping system:

```text
                    Client
                      │
                      ▼
                Load Balancer
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      User Service  Product      Order
                    Service      Service
          │           │           │
          └───────────┼───────────┘
                      ▼
                  Database
```

User Service, Product Service এবং Order Service আলাদা component হতে পারে।

তারা network-এর মাধ্যমে communication করতে পারে।

---

# 14. Cloud এবং Distributed System

একটি গুরুত্বপূর্ণ point:

> **Cloud ব্যবহার করলেই application automatically Distributed System হয়ে যায় না।**

ধরো cloud-এ শুধু একটি VM চালালে:

```text
Cloud
  ↓
1 VM
  ↓
Your Application
```

এটি single-machine application হতে পারে।

অন্যদিকে:

```text
Cloud
  ↓
Server 1
Server 2
Server 3
Database
Cache
Queue
```

এগুলো network-এর মাধ্যমে communicate এবং coordinate করলে এটি distributed architecture হতে পারে।

তাই:

```text
Cloud ≠ Automatically Distributed Application
```

বরং:

> **Cloud distributed architecture তৈরি করার জন্য infrastructure ও services দিতে পারে।**

---

# 15. Process কী?

> **Process হলো একটি running program-এর OS-managed instance।**

ধরো Computer-এ Chrome install করা আছে।

Chrome-এর program file disk-এ রয়েছে।

তুমি Chrome চালু করলে Operating System resources allocate করে এবং running process তৈরি করে।

```text
Program File
     ↓
Run
     ↓
Process
```

---

# 16. Java Program থেকে Process

ধরো:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

Compile করলে:

```text
Hello.java
   ↓
javac
   ↓
Hello.class
```

`Hello.class`-এ Java bytecode থাকে।

তারপর program run করলে JVM application-টিকে execute করে এবং OS-এর অধীনে running process তৈরি হয়।

```text
Hello.java
    ↓
Compile
    ↓
Hello.class
    ↓
Run
    ↓
JVM / Application Execution
    ↓
Process
```

---

# 17. File এবং Process-এর পার্থক্য

## File

File হলো stored data-এর একটি unit।

```text
photo.jpg
video.mp4
Hello.class
program.exe
```

## Process

Process হলো running program-এর execution instance।

```text
Program File
     ↓
Run
     ↓
Process
```

তাই:

```text
.class file ≠ Process
```

বরং:

```text
.class file
     ↓
Running Application
     ↓
Process
```

---

# 18. একটি Machine-এ অনেক Process চলতে পারে

```text
One Computer
├── Chrome Process
├── VS Code Process
├── PostgreSQL Process
├── Spotify Process
└── Java Application Process
```

অনেক process থাকা সত্ত্বেও এটি নিজে থেকে Distributed System নয়।

কারণ:

> সবগুলো process একই machine-এর মধ্যে চলছে।

---

# 19. Multiple Processes ≠ Distributed System

ধরো:

```text
Machine A

Process 1
Process 2
Process 3
Process 4
```

অনেক process আছে।

তবুও:

```text
One Machine
```

তাই এটি Distributed System নয়।

---

# 20. Multiple Machines + Network Communication

এখন:

```text
Machine A
   └── Process A

Machine B
   └── Process B

Machine C
   └── Process C
```

এরা network-এর মাধ্যমে communicate করছে:

```text
Machine A
    ↕
  Network
    ↕
Machine B
    ↕
  Network
    ↕
Machine C
```

এবং যদি তারা coordinate করে common goal অর্জন করে, তাহলে:

> **এটি Distributed System হতে পারে।**

---

# 21. Machine Down হলে কী হয়?

ধরো:

```text
Machine A ✅
Machine B ❌
Machine C ✅
```

যদি system redundancy এবং fault tolerance সহ design করা থাকে, তাহলে Machine A এবং Machine C service চালিয়ে যেতে পারে।

```text
A ✅ + C ✅
     ↓
Service continues
```

এটি Distributed System-এর একটি বড় advantage হতে পারে।

তবে failure handling automatic নয়; system-কে এমনভাবে design করতে হবে যাতে failure handle করতে পারে।

---

# 22. Bitcoin Trading App Example

ধরো তুমি একটি Bitcoin trading mobile app তৈরি করেছো।

শুরুতে:

```text
User
 ↓
Your Home Computer
 ↓
Bitcoin Trading Application
```

১০ জন user → ঠিক আছে।

১০০ user → ঠিক আছে।

১০,০০০ user → load বাড়ছে।

১,০০,০০০ user → একটি machine-এর জন্য অনেক heavy load।

---

## Vertical Scaling

```text
Old Machine
CPU = 4 Core
RAM = 8 GB
```

Upgrade:

```text
New Machine
CPU = 32 Core
RAM = 128 GB
```

এটা **Vertical Scaling**।

---

## Horizontal Scaling

```text
             Users
               │
               ▼
          Load Balancer
        /       |              ▼        ▼        ▼
     Server 1 Server 2 Server 3
```

এটা **Horizontal Scaling**।

Traffic বাড়লে নতুন server যোগ করা যায়:

```text
Server 1
Server 2
Server 3
       ↓
Server 4
Server 5
```

---

# 23. Distributed System কি শুধু "অনেক Server"?

শুধু অনেক server থাকলেই Distributed System বলা ঠিক নয়।

```text
Server 1
Server 2
Server 3
```

যদি তারা completely independent হয় এবং কোনো common distributed task-এ communication বা coordination না করে, তাহলে শুধু "multiple servers" বলা বেশি appropriate।

একটি Distributed System-এর মূল ধারণা:

```text
Multiple Independent Nodes
          +
Network Communication
          +
Coordination
          +
Common Goal
          ↓
Distributed System
```

---

# 24. Centralized বনাম Distributed

## Centralized System

```text
        Users
          │
          ▼
      One Server
          │
          ▼
       Database
```

### বৈশিষ্ট্য

- একটি central machine/system
- তুলনামূলকভাবে সহজ
- সহজে manage করা যায়
- একটি failure বড় সমস্যা করতে পারে
- Scaling limitation বেশি

---

## Distributed System

```text
             Users
               │
               ▼
          Load Balancer
          /     |               ▼      ▼       ▼
        S1     S2      S3
         \      |     /
          \     |    /
           Distributed
             System
```

### বৈশিষ্ট্য

- Multiple machines/components
- Network communication
- Coordination প্রয়োজন
- Better scalability-এর সুযোগ
- Fault tolerance-এর সুযোগ
- Architecture বেশি complex

---

# 25. সহজ Analogy

ধরো একটি restaurant।

## Centralized

একজন মানুষ সব কাজ করছে:

```text
One Person
├── Order নেয়
├── রান্না করে
├── Payment নেয়
└── Delivery দেয়
```

সে unavailable হলে সব কাজ বন্ধ।

## Distributed

```text
Waiter
   ↓
Chef
   ↓
Cashier
   ↓
Delivery
```

প্রত্যেকের আলাদা responsibility আছে এবং তারা communication করে।

এটি Distributed System বোঝার একটি সহজ analogy।

---

# 26. সবচেয়ে গুরুত্বপূর্ণ Concept

```text
File
↓
Stored Data

Program
↓
Code / Instructions

Process
↓
Running Program

One Machine + Many Processes
↓
Distributed System নয়

Multiple Machines
+
Network Communication
+
Coordination
+
Common Goal
↓
Distributed System
```

---

# 27. Exam-এর জন্য Definition

## Distributed System — English

> A distributed system is a collection of independent computers or processes that communicate and coordinate with each other over a network to achieve a common goal and provide a unified service.

## Distributed System — বাংলা

> **Distributed System হলো একাধিক independent computer বা process-এর সমষ্টি, যারা network-এর মাধ্যমে একে অপরের সাথে communication ও coordination করে একটি common goal অর্জন করে এবং user-এর কাছে একটি unified service প্রদান করে।**

---

# 28. Quick Revision

## Distributed System

```text
Multiple Machines
+
Network
+
Communication
+
Coordination
+
Common Goal
```

## Centralized System

```text
One Main Machine/System
```

## Vertical Scaling

```text
One Machine
      ↓
More CPU/RAM
      ↓
More Powerful Machine
```

## Horizontal Scaling

```text
One Machine
      ↓
Multiple Machines
      ↓
More Servers
```

## Process

```text
Program
   ↓
Running
   ↓
Process
```

## File

```text
Stored Data
```

---

# 29. One-Minute Mental Model

```text
                    USERS
                      │
                      ▼
               WEBSITE / APP
                      │
                      ▼
                Load Balancer
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Server 1     Server 2    Server 3
          │           │           │
          └───────────┼───────────┘
                      │
                   Network
                      │
              Communication +
                Coordination
                      │
                      ▼
                   Database
```

User-এর কাছে এটি একটি website/app-এর মতো মনে হলেও backend-এ একাধিক machine ও component একসাথে কাজ করতে পারে।

---

# 🔑 Final Memory Trick

> **Vertical Scaling = একটি machine বড় করো।**  
> **Horizontal Scaling = আরও machine যোগ করো।**  
> **Process = Running program।**  
> **File = Stored data।**  
> **Multiple processes on one machine ≠ Distributed System।**  
> **Multiple machines + network communication + coordination + common goal = Distributed System।**

## ✅ সবচেয়ে গুরুত্বপূর্ণ লাইন

> **Distributed System-এর আসল idea শুধু "অনেক computer" নয়; বরং অনেক independent machine/process network-এর মাধ্যমে communicate এবং coordinate করে user-এর কাছে একটি unified service প্রদান করে।**

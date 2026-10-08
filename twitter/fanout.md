# 🐦 Twitter System Design — Fanout & Timeline

Twitter-এর architecture থেকে **Fanout** concept বোঝা যায় খুব ভালোভাবে। এখানে মূল challenge ছিল tweet লেখা নয়, বরং বিশাল সংখ্যক user-এর জন্য homepage timeline খুব দ্রুত দেখানো।

---

## 1. Twitter-এর মূল সমস্যা

২০১২–২০১৩ সালের দিকে Twitter প্রায়:

- **150 million users**
- প্রায় **6,000 tweets/second**

handle করছিল।

Tweet **write throughput** manageable ছিল, কিন্তু বড় challenge ছিল **reading tweets**।

কারণ homepage query করার সময় প্রায়:

**300,000 read requests/second**

হতো।

```mermaid
flowchart LR
    A[150M Users] --> B[Twitter]
    B --> C[~6,000 Tweets/sec]
    B --> D[~300,000 Homepage Reads/sec]

    classDef users fill:#4F46E5,color:#fff,stroke:#312E81;
    classDef twitter fill:#06B6D4,color:#fff,stroke:#155E75;
    classDef write fill:#22C55E,color:#fff,stroke:#166534;
    classDef read fill:#F97316,color:#fff,stroke:#9A3412;

    class A users;
    class B twitter;
    class C write;
    class D read;
```

---

## 2. User Timeline vs Home Timeline

Twitter দুই ধরনের timeline আলাদা করেছিল।

### 👤 User Timeline

এখানে একজন user-এর **নিজের tweets** থাকে।

```text
Rahim
  ↓
His Tweets
  ↓
User Timeline
```

### 🏠 Home Timeline

এখানে user যাদের follow করে, তাদের tweets থাকে।

```text
Rahim follows:
├── Karim
├── Sakib
└── Nabila

        ↓

Rahim's Home Timeline
= Karim + Sakib + Nabila-এর tweets
```

Mermaid:

```mermaid
flowchart TB
    U[User] --> F[People User Follows]

    F --> A[Person A Tweets]
    F --> B[Person B Tweets]
    F --> C[Person C Tweets]

    A --> H[Home Timeline]
    B --> H
    C --> H

    classDef user fill:#4F46E5,color:#fff,stroke:#312E81;
    classDef follow fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef home fill:#F59E0B,color:#fff,stroke:#92400E;

    class U user;
    class F follow;
    class A,B,C tweet;
    class H home;
```

---

# 3. প্রথমে কী ভাবা হয়েছিল?

প্রথমে simple database approach চিন্তা করা হয়েছিল।

User-এর timeline-এর জন্য database-এ tweets append করা এবং indexes ব্যবহার করা যায়।

কিন্তু এত বেশি **read volume**-এর জন্য এই approach যথেষ্ট fast ছিল না।

```mermaid
flowchart LR
    T[New Tweet] --> DB[Database]
    DB --> I[Indexes]
    I --> R[Homepage Read]

    X[High Read Volume] --> S[Too Slow]

    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef db fill:#3B82F6,color:#fff,stroke:#1E3A8A;
    classDef index fill:#8B5CF6,color:#fff,stroke:#5B21B6;
    classDef read fill:#F59E0B,color:#fff,stroke:#92400E;
    classDef slow fill:#EF4444,color:#fff,stroke:#991B1B;

    class T tweet;
    class DB db;
    class I index;
    class R read;
    class X,S slow;
```

---

# 4. Twitter-এর সমাধান: Pre-computed Home Timeline

Twitter-এর approach ছিল:

**Home Timeline আগে থেকেই তৈরি করে রাখা।**

এজন্য **Redis cluster** ব্যবহার করা হয়েছিল।

অর্থাৎ user যখন Twitter খুলবে, তখন নতুন করে সব tweets খুঁজে timeline বানানোর পরিবর্তে আগে থেকেই তৈরি করা timeline থেকে data নেওয়া যাবে।

```mermaid
flowchart LR
    T[New Tweet] --> F[Fanout Process]
    F --> R[Redis Cluster]
    R --> H[Pre-computed Home Timeline]
    H --> U[User Reads Timeline]

    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef fanout fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef redis fill:#EF4444,color:#fff,stroke:#991B1B;
    classDef timeline fill:#F59E0B,color:#fff,stroke:#92400E;
    classDef user fill:#3B82F6,color:#fff,stroke:#1E3A8A;

    class T tweet;
    class F fanout;
    class R redis;
    class H timeline;
    class U user;
```

---

# 5. Fanout কী?

**Fanout** হলো:

কেউ tweet করলে, সেই tweet-এর copy তার followers-দের **Home Timeline**-এ পাঠিয়ে রাখা।

ধরো:

```text
Rahim tweets
     ↓
   Fanout
     ↓
+----+----+----+----+
|    |    |    |    |
U1   U2   U3   ... U1000
```

অর্থাৎ Rahim-এর 1,000 followers থাকলে tweet করার সময় tweet-টি তাদের Home Timeline-এর জন্য replicate করা হবে।

```mermaid
flowchart TB
    T[User Tweets] --> F[Fanout]

    F --> U1[Follower 1]
    F --> U2[Follower 2]
    F --> U3[Follower 3]
    F --> U4[...]

    U1 --> H1[Home Timeline]
    U2 --> H2[Home Timeline]
    U3 --> H3[Home Timeline]
    U4 --> H4[Home Timeline]

    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef fanout fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef follower fill:#3B82F6,color:#fff,stroke:#1E3A8A;
    classDef home fill:#F59E0B,color:#fff,stroke:#92400E;

    class T tweet;
    class F fanout;
    class U1,U2,U3,U4 follower;
    class H1,H2,H3,H4 home;
```

---

# 6. Fanout-এর মূল Trade-off

Fanout ব্যবহার করলে **writes বেড়ে যায়**।

কারণ একটি tweet শুধু এক জায়গায় রাখলেই হবে না; followers-এর Home Timeline-এও সেটি replicate করতে হবে।

কিন্তু এর বিনিময়ে **read latency অনেক কমে যায়**।

### সহজভাবে:

```text
More Writes
     ↓
Pre-computed Timeline
     ↓
Much Faster Reads
```

```mermaid
flowchart LR
    A[One Tweet] --> B[Fanout]
    B --> C[More Writes]
    B --> D[Pre-computed Home Timelines]
    D --> E[Much Faster Reads]

    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef fanout fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef writes fill:#EF4444,color:#fff,stroke:#991B1B;
    classDef timeline fill:#F59E0B,color:#fff,stroke:#92400E;
    classDef reads fill:#06B6D4,color:#fff,stroke:#155E75;

    class A tweet;
    class B fanout;
    class C writes;
    class D timeline;
    class E reads;
```

---

# 7. High-profile User-এর সমস্যা

এবার ধরো **Lady Gaga**-এর মতো একজন high-profile user-এর **millions of followers** আছে।

সে যদি একটা tweet করে এবং normal fanout করা হয়:

```text
1 Tweet
   ↓
Millions of Followers
   ↓
Millions of Timeline Updates
```

এতে massive fanout delay হতে পারে।

তাই high-profile users-এর জন্য **hybrid approach** ব্যবহার করা হয়।

তাদের tweets সব followers-এর timeline-এ massiveভাবে আগে থেকে fanout না করে, **query time-এ merge** করা হয়।

```mermaid
flowchart TB
    A[High-profile User Tweets] --> B[Hybrid Approach]
    B --> C[No Massive Fanout]
    B --> D[Merge at Query Time]

    E[User Opens Twitter] --> D
    D --> F[Home Timeline]

    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef hybrid fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef nofanout fill:#EF4444,color:#fff,stroke:#991B1B;
    classDef query fill:#F59E0B,color:#fff,stroke:#92400E;
    classDef home fill:#3B82F6,color:#fff,stroke:#1E3A8A;

    class A tweet;
    class B hybrid;
    class C nofanout;
    class D,E query;
    class F home;
```

---

# 8. Normal User বনাম High-profile User

```mermaid
flowchart TB
    T1[Normal User Tweet] --> F1[Fanout]
    F1 --> H1[Followers' Home Timelines]

    T2[High-profile User Tweet] --> F2[Hybrid Approach]
    F2 --> Q[Merge at Query Time]
    Q --> H2[Home Timeline]

    classDef normal fill:#22C55E,color:#fff,stroke:#166534;
    classDef high fill:#EF4444,color:#fff,stroke:#991B1B;
    classDef fanout fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef query fill:#F59E0B,color:#fff,stroke:#92400E;
    classDef home fill:#3B82F6,color:#fff,stroke:#1E3A8A;

    class T1 normal;
    class T2 high;
    class F1 fanout;
    class F2,Q query;
    class H1,H2 home;
```

---

# 9. 2022 সালের Architecture

২০২২ সালের দিকে এই architecture **230 million+ users** serve করছিল।

এখানে লক্ষ্য ছিল একটি message যেন একজন user-এর কাছে **5 seconds-এর মধ্যে** পৌঁছে যায়।

Transcript অনুযায়ী flow-এ ছিল:

```text
Load Balancer
      ↓
Flock
      ↓
Redis
      ↓
Pre-computed Cache
      ↓
User
```

এখানে **Flock** হলো Twitter-এর একটি **social graph service**।

```mermaid
flowchart LR
    A[User] --> B[Load Balancer]
    B --> C[Flock]
    C --> D[Redis]
    D --> E[Pre-computed Cache]
    E --> F[User Timeline]

    classDef user fill:#3B82F6,color:#fff,stroke:#1E3A8A;
    classDef lb fill:#06B6D4,color:#fff,stroke:#155E75;
    classDef flock fill:#8B5CF6,color:#fff,stroke:#5B21B6;
    classDef redis fill:#EF4444,color:#fff,stroke:#991B1B;
    classDef cache fill:#F59E0B,color:#fff,stroke:#92400E;

    class A,F user;
    class B lb;
    class C flock;
    class D redis;
    class E cache;
```

---

# 10. পুরো Fanout Flow

একজন normal user tweet করলে পুরো concept:

```mermaid
flowchart TB
    T[User Creates Tweet] --> F[Fanout Process]

    F --> R[Replicate Tweet]

    R --> H1[Follower 1 Home Timeline]
    R --> H2[Follower 2 Home Timeline]
    R --> H3[Follower 3 Home Timeline]
    R --> H4[More Followers]

    H1 --> RC[Redis Cluster]
    H2 --> RC
    H3 --> RC
    H4 --> RC

    RC --> U[Fast Home Timeline Read]

    classDef tweet fill:#22C55E,color:#fff,stroke:#166534;
    classDef fanout fill:#A855F7,color:#fff,stroke:#6B21A8;
    classDef replicate fill:#F97316,color:#fff,stroke:#9A3412;
    classDef timeline fill:#3B82F6,color:#fff,stroke:#1E3A8A;
    classDef redis fill:#EF4444,color:#fff,stroke:#991B1B;
    classDef read fill:#06B6D4,color:#fff,stroke:#155E75;

    class T tweet;
    class F fanout;
    class R replicate;
    class H1,H2,H3,H4 timeline;
    class RC redis;
    class U read;
```

---

# 11. Fanout কেন ব্যবহার করা হলো?

মূল কারণ ছিল **read performance**।

```text
Database-based approach
        ↓
High Read Volume
        ↓
Too Slow


Fanout + Pre-computed Timeline
        ↓
More Writes
        ↓
Much Faster Reads
```

অর্থাৎ Twitter-এর design-এর মূল idea:

> **Write একটু বেশি করা → Read অনেক দ্রুত করা**

---

# 12. 🧠 Easy Memory Trick

### **Fanout = Tweet ছড়িয়ে দেওয়া**

মনে রাখো:

```text
1 Person Tweets
       ↓
Fanout
       ↓
Many Followers
       ↓
Their Home Timelines
       ↓
Fast Read
```

### এক লাইনে:

**“User tweet করলে Fanout সেই tweet-এর copy followers-এর Home Timeline-এ আগে থেকেই রেখে দেয়, যাতে পরে timeline পড়া দ্রুত হয়।”**

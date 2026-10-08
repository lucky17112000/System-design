# 🚦 Rate Limiting — System Design Notes (বাংলা)

## 1. Rate Limiting কী?

**Rate Limiting** হলো কোনো service-এ নির্দিষ্ট সময়ের মধ্যে কতগুলো operation বা request করা যাবে, তার limit নির্ধারণ করা। এটি system overload এবং denial-of-service (DoS)-এর মতো malicious attack-এর ঝুঁকি কমাতে সাহায্য করে।

Threshold অতিক্রম করলে window reset হওয়া পর্যন্ত client-কে error response দেওয়া হতে পারে।

```mermaid
flowchart TD
    A["Client Request"] --> B["Rate Limiter"]
    B --> C{"Threshold-এর মধ্যে?"}
    C -->|হ্যাঁ| D["Request Allow"]
    C -->|না| E["Error Response"]
    classDef a fill:#2563EB,color:#fff,stroke:#1E40AF
    classDef b fill:#7C3AED,color:#fff,stroke:#5B21B6
    classDef c fill:#F59E0B,color:#111827,stroke:#B45309
    classDef d fill:#16A34A,color:#fff,stroke:#166534
    classDef e fill:#DC2626,color:#fff,stroke:#991B1B
    class A a
    class B b
    class C c
    class D d
    class E e
```

## 2. Rate Limiting-এর Strategies

Transcript-এ তিনটি strategy উল্লেখ করা হয়েছে:

- **User-based limiting:** নির্দিষ্ট user-এর request limit করা।
- **Geographic-based filtering:** ভৌগোলিক অবস্থানের ভিত্তিতে request filter করা।
- **Server-based limiting:** server-এর ভিত্তিতে request limit করা।

```mermaid
flowchart TD
    A["Rate Limiting Strategies"] --> B["User-based"]
    A --> C["Geographic-based"]
    A --> D["Server-based"]
    classDef root fill:#7C3AED,color:#fff,stroke:#5B21B6
    classDef user fill:#2563EB,color:#fff,stroke:#1E40AF
    classDef geo fill:#0891B2,color:#fff,stroke:#155E75
    classDef server fill:#16A34A,color:#fff,stroke:#166534
    class A root
    class B user
    class C geo
    class D server
```

## 3. Fixed Window

**Fixed Window** নির্দিষ্ট সময়ের window-তে request count করে এবং একটি limit বজায় রাখে।

উদাহরণ: একটি window-তে সর্বোচ্চ 5টি request অনুমোদন করা।

```mermaid
flowchart TD
    A["Incoming Request"] --> B["Fixed Time Window"]
    B --> C["Request Count"]
    C --> D{"Limit-এর মধ্যে?"}
    D -->|হ্যাঁ| E["Request Allow"]
    D -->|না| F["Request Reject"]
    classDef a fill:#2563EB,color:#fff,stroke:#1E40AF
    classDef b fill:#7C3AED,color:#fff,stroke:#5B21B6
    classDef c fill:#F59E0B,color:#111827,stroke:#B45309
    classDef d fill:#16A34A,color:#fff,stroke:#166534
    classDef e fill:#DC2626,color:#fff,stroke:#991B1B
    class A a
    class B,C b
    class D c
    class E d
    class F e
```

**অসুবিধা:** Window-এর শেষের দিকে অনেক request এবং পরের window-এর শুরুতে আবার অনেক request এলে burst তৈরি হতে পারে।

## 4. Sliding Window

**Sliding Window** Fixed Window-এর approach-কে refine করে, যাতে traffic সময়ের সঙ্গে আরও smoothly handle করা যায়।

```mermaid
flowchart TD
    A["Incoming Request"] --> B["Sliding Window"]
    B --> C["সময়ের ভিত্তিতে Request হিসাব"]
    C --> D{"Limit-এর মধ্যে?"}
    D -->|হ্যাঁ| E["Request Allow"]
    D -->|না| F["Request Reject"]
    classDef a fill:#2563EB,color:#fff,stroke:#1E40AF
    classDef b fill:#7C3AED,color:#fff,stroke:#5B21B6
    classDef c fill:#F59E0B,color:#111827,stroke:#B45309
    classDef d fill:#16A34A,color:#fff,stroke:#166534
    classDef e fill:#DC2626,color:#fff,stroke:#991B1B
    class A a
    class B,C b
    class D c
    class E d
    class F e
```

**মূল কথা:** Fixed Window নির্দিষ্ট time block ব্যবহার করে; Sliding Window সময়ের সঙ্গে হিসাবকে আরও smoothly পরিচালনা করে।

## 5. Token Bucket

**Token Bucket**-এ bucket-এর মধ্যে token থাকে। Request process করতে একটি token প্রয়োজন হয়।

- Token থাকলে request process হয়।
- Token না থাকলে request drop করা বা queue-তে রাখা হয়।

```mermaid
flowchart TD
    A["Incoming Request"] --> B["Token Bucket"]
    B --> C{"Token Available?"}
    C -->|হ্যাঁ| D["Token ব্যবহার"]
    D --> E["Request Process"]
    C -->|না| F["Request Drop অথবা Queue"]
    classDef a fill:#2563EB,color:#fff,stroke:#1E40AF
    classDef b fill:#7C3AED,color:#fff,stroke:#5B21B6
    classDef c fill:#F59E0B,color:#111827,stroke:#B45309
    classDef d fill:#16A34A,color:#fff,stroke:#166534
    classDef e fill:#DC2626,color:#fff,stroke:#991B1B
    class A a
    class B b
    class C c
    class D,E d
    class F e
```

**মনে রাখো:** Token হলো request process করার অনুমতির মতো।

## 6. Leaky Bucket

**Leaky Bucket** bursty traffic-কে একটি নির্দিষ্ট rate-এ বের করে, যাতে request-এর flow uniform হয়।

```mermaid
flowchart TD
    A["Bursty Incoming Traffic"] --> B["Leaky Bucket"]
    B --> C["Fixed Rate-এ Request বের হয়"]
    C --> D["Uniform Traffic Flow"]
    classDef a fill:#F97316,color:#fff,stroke:#C2410C
    classDef b fill:#7C3AED,color:#fff,stroke:#5B21B6
    classDef c fill:#2563EB,color:#fff,stroke:#1E40AF
    classDef d fill:#16A34A,color:#fff,stroke:#166534
    class A a
    class B b
    class C c
    class D d
```

## 7. চারটি Algorithm-এর তুলনা

| Algorithm          | মূল ধারণা                           | গুরুত্বপূর্ণ বিষয়                  |
| ------------------ | ----------------------------------- | ---------------------------------- |
| **Fixed Window**   | নির্দিষ্ট সময়ের মধ্যে request count | Window boundary-তে burst হতে পারে  |
| **Sliding Window** | সময়ের সঙ্গে request হিসাব           | Traffic আরও smoothly handle করে    |
| **Token Bucket**   | Request-এর জন্য token দরকার         | Token না থাকলে drop বা queue       |
| **Leaky Bucket**   | Fixed rate-এ request বের হয়         | Bursty traffic-কে uniform flow করে |

## 8. 🧠 Quick Memory Trick

- **Fixed Window** → নির্দিষ্ট সময়ের window
- **Sliding Window** → সময়ের সঙ্গে sliding হিসাব
- **Token Bucket** → Token থাকলে request process
- **Leaky Bucket** → Fixed rate-এ request flow

### Interview Answer

**Rate Limiting** হলো নির্দিষ্ট সময়ের মধ্যে service-এ করা request বা operation-এর সংখ্যা নিয়ন্ত্রণ করার পদ্ধতি। এটি system overload এবং DoS-এর মতো malicious attack-এর ঝুঁকি কমাতে সাহায্য করে। Common algorithms হলো **Fixed Window, Sliding Window, Token Bucket এবং Leaky Bucket**।

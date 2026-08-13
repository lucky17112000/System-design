# 🗄️ Database — System Design Notes

## 1. Database-এর মূল ২টি কাজ

একটি Database-এর প্রধান উদ্দেশ্য মূলত দুইটি:

> **1. Data Store করা**  
> **2. Data Retrieve করা**

অর্থাৎ:

```text
        DATABASE
        /      \
       ↓        ↓
   STORE      RETRIEVE
   Data        Data
```

### Store

Database-এ নতুন data রাখা।

উদাহরণ:

```text
User:
Name = Rahim
Age = 25
Email = rahim@gmail.com
```

এই information Database-এ save করা হয়।

### Retrieve

Database থেকে প্রয়োজনীয় data বের করে আনা।

যেমন:

```sql
SELECT * FROM users
WHERE id = 101;
```

Database তখন User ID 101-এর information return করবে।

---

# 2. Database কি শুধু একটা Software?

এখানে একটি important বিষয় বুঝতে হবে।

অনেকে ভাবে:

> "Database মানে একটা software."

আসলে Database System-এর মধ্যে কয়েকটি related জিনিস থাকতে পারে:

```text
Database System
├── Data
├── Database Software
├── Storage
└── Computing Resources
```

Database software-এর উদাহরণ:

- PostgreSQL
- MySQL
- MongoDB
- Microsoft SQL Server
- Oracle Database

---

# 3. Database কি একটা Server?

তুমি যদি বলো:

> **"A database is just a server."**

এটা পুরোপুরি accurate নয়।

বরং বলবে:

> **Database হলো data store, manage এবং retrieve করার জন্য ব্যবহৃত একটি software/system, যা একটি server machine-এ run করতে পারে।**

উদাহরণ:

```text
Computer
   │
   ├── Operating System
   │
   ├── PostgreSQL
   │
   └── Database
```

এখানে:

- **Computer/Server** = Physical বা virtual machine
- **PostgreSQL** = Database Management System (DBMS)
- **Database** = Structured data-এর collection

---

# 4. যেকোনো Computer কি Database Server হতে পারে?

**হ্যাঁ, technically পারে।**

একটি laptop-এ তুমি PostgreSQL install করলে:

```text
Your Laptop
     │
     └── PostgreSQL
             │
             └── Database
```

এখন তোমার laptop Database Server হিসেবে কাজ করতে পারে।

ধরো তোমার laptop-এ:

```text
PostgreSQL
     ↓
users table
     ↓
products table
     ↓
orders table
```

এখন অন্য application তোমার laptop-এর PostgreSQL-এর সাথে connect করে data store এবং retrieve করতে পারে।

---

# 5. কিন্তু Laptop আর Database একই জিনিস নয়

এটা খুব ভালোভাবে মনে রাখবে।

ধরো:

```text
Laptop
   ↓
PostgreSQL
   ↓
Database
```

এখানে:

### Laptop

**Physical Machine**

### PostgreSQL

**Database Management Software / DBMS**

### Database

**Stored and organized data**

তাই:

> **Laptop নিজে Database নয়; Laptop-এর মধ্যে Database Software চালিয়ে Database System তৈরি করা যায়।**

---

# 6. একটি সহজ Real-Life Example

ধরো তোমার laptop-এ PostgreSQL install করেছো।

তুমি command দিলে:

```sql
CREATE DATABASE university;
```

তারপর:

```sql
CREATE TABLE students (
    id INT,
    name VARCHAR(100),
    department VARCHAR(50)
);
```

তারপর data store করলে:

```sql
INSERT INTO students
VALUES (1, 'Rahim', 'CSE');
```

এখন Database-এ data আছে:

```text
University Database
        │
        ▼
Students Table
        │
        └── 1 | Rahim | CSE
```

তারপর তুমি data retrieve করতে পারো:

```sql
SELECT * FROM students;
```

Result:

```text
1 | Rahim | CSE
```

---

# 7. Database-এর মূল কাজ

Database-এর basic কাজ:

```text
              DATABASE
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
      STORE              RETRIEVE
        │                   │
        ↓                   ↓
   Save Data            Get Data
```

তবে বাস্তবে Database আরও অনেক কাজ করে:

- Store
- Retrieve
- Update
- Delete
- Search
- Indexing
- Concurrency Control
- Security
- Backup and Recovery
- Transaction Management

তাই শুধু **Store + Retrieve** হলো মূল উদ্দেশ্য; Database-এর কাজ এর চেয়ে অনেক বেশি।

---

# ⭐ Important Concept

তুমি যদি exam বা interview-এ জিজ্ঞেস করা হয়:

### "What is a Database?"

তাহলে বলতে পারো:

> **A database is a software/system used to store, manage, and retrieve data efficiently. It can run on a physical or virtual server.**

বাংলায়:

> **Database হলো data store, manage এবং retrieve করার জন্য ব্যবহৃত একটি software/system, যা physical বা virtual server-এর উপর run করতে পারে।**

---

# 🧠 সহজে মনে রাখো

```text
Database
   =
Store + Retrieve + Manage Data
```

আর:

```text
Database Server
   =
Computer/VM
   +
Database Software
```

### সবচেয়ে গুরুত্বপূর্ণ পার্থক্য

```text
Laptop/Server
     ↓
Physical/Virtual Machine

PostgreSQL/MySQL
     ↓
Database Software / DBMS

Database
     ↓
Stored & Organized Data
```

> **একটি Laptop Database নয়। Laptop-এর মধ্যে PostgreSQL/MySQL-এর মতো Database Software চালিয়ে সেটিকে Database Server হিসেবে ব্যবহার করা যায়।**

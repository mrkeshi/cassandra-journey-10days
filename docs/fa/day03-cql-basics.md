# روز سوم – شروع با CQL
![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white) 
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)

قبل از ورود به **CQL (زبان پرس‌وجوی کاساندرا)**، بهتر است چند نکته را در نظر داشته باشید:

- **کاساندرا از مقادیر `DEFAULT` پشتیبانی نمی‌کند.**  
  اگر هنگام درج (insert) مقداری ارائه نشود، آن مقدار به‌سادگی **حذف می‌شود** — رفتاری مانند پایگاه‌های داده سنتی ندارد.

- **کاساندرا سریع، ساده و بسیار مقیاس‌پذیر است،** اما از **تمام ویژگی‌های پایگاه‌های داده رابطه‌ای** (مانند `DEFAULT`، `NOT NULL`) **پشتیبانی نمی‌کند.**  
  بنابراین، **اعتبارسنجی داده‌ها و مدیریت مقادیر پیش‌فرض** باید در **سطح برنامه کاربردی** انجام شود.

- در کاساندرا، **`NULL` در واقع به معنای تعیین نشدن ستون است (یعنی ذخیره نشده).**  
  این متفاوت از `NULL` در SQL است — بیشتر شبیه این است که ستون برای آن ردیف **وجود ندارد.**

✅ در بخش‌های بعدی، مجموعه کاملی از عملیات‌های **CRUD (ایجاد، خواندن، به‌روزرسانی، حذف)** با استفاده از CQL را پوشش خواهیم داد.


## ✍️ ایجاد جدول و کی‌اسپیس

📥 در این بخش، نحوه **ایجاد یک کی‌اسپیس**، تعریف **جداول کاساندرا** و درک **ساختار کلیدهای اصلی (primary key)** را بررسی می‌کنیم.

---

### 🔹 ایجاد یک کی‌اسپیس

*کی‌اسپیس* در کاساندرا مشابه پایگاه داده در سیستم‌های رابطه‌ای است. این عنصر تعریف می‌کند که چگونه داده‌ها در بین گره‌ها تکرار و ذخیره می‌شوند.

```sql
CREATE KEYSPACE IF NOT EXISTS my_keyspace 
WITH replication = {
  'class': 'SimpleStrategy', 
  'replication_factor': 1
};
AND durable_writes = true;
```

```
USE my_keyspace;
```

- `USE my_keyspace`  این فرمان کی‌اسپیس جاری را برای نشست شما تنظیم می‌کند، بنابراین تمام دستورات CQL بعدی در این کی‌اسپیس اجرا می‌شوند مگر اینکه به طور خاص مشخص شود.
-  `durable_writes` (پیش‌فرض: true) تضمین می‌کند که نوشتن‌ها ابتدا در لاگ ثبت (commit log) روی دیسک ذخیره شوند.
- `SimpleStrategy` فقط برای محیط‌های تک‌گره یا توسعه پیشنهاد می‌شود.  
- `replication_factor` مشخص می‌کند که چند نسخه از داده‌ها ذخیره شود. برای محیط تولیدی از `NetworkTopologyStrategy` استفاده کنید.

---

### 🔹 ایجاد جدول

بیایید یک جدول `users` با انواع داده مختلف ایجاد کنیم:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    age INT,
    is_active BOOLEAN,
    signup_date TIMESTAMP,
    interests LIST<TEXT>,
    settings MAP<TEXT, TEXT>
);
```

#### 🔸 انواع فیلدهای استفاده‌شده:

| نوع        | توضیح                                    |
|------------|-------------------------------------------|
| `UUID`     | شناسه یکتا جهانی                         |
| `TEXT`     | رشته‌ای با کدگذاری UTF-8                |
| `INT`      | عدد صحیح                                  |
| `BOOLEAN`  | درست/نادرست                              |
| `TIMESTAMP`| تاریخ و زمان                              |
| `LIST<T>`  | مجموعه مرتب‌شده از عناصر (تکراری مجاز است) |
| `MAP<K,V>` | ساختار کلید-مقدار                         |

> 💡 کاساندرا همچنین از `SET<TEXT>` پشتیبانی می‌کند (مانند `LIST` اما بدون ترتیب و بدون تکرار).

---

### 🔹 درک کلید اصلی (PRIMARY KEY)

`PRIMARY KEY` در کاساندرا تعیین‌کننده **توزیع داده‌ها** و **ترتیب‌گذاری (clustering)** درون پارتیشن‌ها است.

```sql
PRIMARY KEY ((ستون‌های کلید پارتیشن), ستون‌های کلسترینگ)
```

#### ✔ پاسخ کوتاه:

- کاساندرا فقط اجازه **یک کلید پارتیشن** را می‌دهد.  
- اما این کلید می‌تواند **ترکیبی** از **چند ستون** باشد.

#### 🧠 توضیح کامل:

`PRIMARY KEY` به دو بخش تقسیم می‌شود:

- **کلید پارتیشن** → تعیین می‌کند داده در *کدام گره* ذخیره شود.
- **ستون‌های کلسترینگ** → نحوه *مرتب‌سازی داده‌ها* درون پارتیشن را مشخص می‌کند.

**مثال:**

```sql
CREATE TABLE example (
    country TEXT,
    city TEXT,
    name TEXT,
    age INT,
    PRIMARY KEY ((country, city), name)
);
```

- **کلید پارتیشن:** `(country, city)` → داده‌ها بر اساس این کلید ترکیبی پارتیشن‌بندی می‌شوند.
- **کلید کلسترینگ:** `name` → داده‌ها بر اساس نام در هر پارتیشن مرتب می‌شوند.

بنابراین همه ردیف‌هایی که `country` و `city` یکسان دارند، با هم ذخیره می‌شوند و بر اساس `name` مرتب می‌شوند.

---

### ⚠️ حساسیت به حروف کوچک/بزرگ در کاساندرا

کاساندرا به‌طور پیش‌فرض **نسبت به حروف کوچک و بزرگ حساس نیست:**

```sql
SELECT name FROM users;
SELECT NAME FROM USERS;
SELECT NaMe FROM UsErS;
```

همه موارد بالا یکسان در نظر گرفته می‌شوند.

> ❗ اگر از کوتیشن دوتایی (مثلاً `"Name"`) استفاده کنید، شناسه **حساس به حروف** می‌شود. مگر در مواقع ضروری از کوتیشن استفاده نکنید.

**مثال:**

```sql
CREATE TABLE "USERS" (
    "Name" TEXT PRIMARY KEY,
    age INT
);

-- این درست کار می‌کند:
SELECT "Name" FROM "USERS";

-- این موارد خطا می‌دهند چون شناسه‌ها در حالت کوتیشن حساس به حروف هستند:
SELECT name FROM users;
SELECT "name" FROM "USERS";
SELECT "Name" FROM users;
```

---

## ✍️ وارد کردن داده در کاساندرا

📥 در این بخش، نحوه **وارد کردن داده به جداول** با استفاده از CQL را بررسی می‌کنیم؛ شامل درج تک‌ردیفی و درج گروهی.

---

### 🔹 درج ساده (Basic INSERT)

می‌توانید یک ردیف را با دستور `INSERT INTO` وارد کنید:

```sql
INSERT INTO users (
    id, name, email, age, is_active, signup_date, interests, settings
) VALUES (
    uuid(), 'Ali', 'ali@example.com', 30, true, toTimestamp(now()), ['coding', 'reading'], {'theme': 'dark', 'lang': 'fa'}
);
```

- `uuid()` یک شناسه منحصربه‌فرد ایجاد می‌کند.
- `toTimestamp(now())` تاریخ و زمان فعلی را وارد می‌کند.
- مقادیر `LIST` و `MAP` با `[ ]` و `{ }` قابل استفاده‌اند.

---

### 🔹 درج ناقص (Partial INSERT)

نیازی به تعیین تمام ستون‌ها نیست — فقط ستون‌هایی که می‌خواهید مقداردهی کنید:

```sql
INSERT INTO users (id, name, email) 
VALUES (uuid(), 'Sara', 'sara@example.com');
```

ستون‌های نامشخص (مانند `age`، `signup_date` و ...) به عنوان **unset** در نظر گرفته می‌شوند — ذخیره نمی‌گردند.

---

### 🔹 درج گروهی (BATCH INSERT)

برای گروه‌بندی چند دستور درج از `BEGIN BATCH` استفاده کنید:

```sql
BEGIN BATCH
INSERT INTO users (id, name, email, age) VALUES (uuid(), 'Reza', 'reza@example.com', 25);
INSERT INTO users (id, name, email, age) VALUES (uuid(), 'Niloofar', 'niloofar@example.com', 28);
INSERT INTO users (id, name, email, age) VALUES (uuid(), 'Mehdi', 'mehdi@example.com', 31);
APPLY BATCH;
```

> ⚠️ BATCH برای درج اتمی **در یک پارتیشن** مفید است. از درج‌های بزرگ یا بین‌پارتیشنی خودداری کنید.

---

### 🔎 جدول نمونه پس از درج

پس از درج رکوردها، کوئری مانند زیر:

```sql
SELECT * FROM users;
```

ممکن است نتیجه‌ای مشابه بازگرداند:

| id                                   | name     | email               | age | is_active | signup_date         | interests             | settings                         |
|--------------------------------------|----------|---------------------|-----|-----------|----------------------|------------------------|-----------------------------------|
| 550e8400-e29b-41d4-a716-446655440000 | Ali      | ali@example.com     | 30  | true      | 2025-07-30 14:20:15  | ['coding', 'reading']  | {'theme': 'dark', 'lang': 'fa'}  |
| 8b3c8d2e-3d47-4ae1-9936-4ed2ee7a1234 | Sara     | sara@example.com    | null| null      | null                 | null                   | null                              |
| 72f8d9f7-9841-4d35-9a15-47f8a7e9b567 | Reza     | reza@example.com    | 25  | null      | null                 | null                   | null                              |
| 4f2a1c17-6c6b-4d2d-a1d2-1a2f3d45e123 | Niloofar | niloofar@example.com| 28  | null      | null                 | null                   | null                              |
| 1a2b3c4d-5e6f-7a8b-9c0d-112233445566 | Mehdi    | mehdi@example.com   | 31  | null      | null                 | null                   | null     |

> 💡 از `SELECT column1, column2 FROM users;` برای دریافت فقط برخی ستون‌ها استفاده کنید.

---

### ✅ نکات

- `INSERT INTO` برای اضافه کردن ردیف‌های جدید به جدول استفاده می‌شود.
- می‌توانید ردیف کامل یا ناقص وارد کنید.
- از `BEGIN BATCH` برای درج چندین ردیف به صورت گروهی بهره ببرید.
- کاساندرا از auto-increment یا مقدار پیش‌فرض پشتیبانی نمی‌کند — این منطق باید از سمت برنامه اعمال شود.

---

> ❗ کاساندرا از درج چند ردیف به صورت SQL-style پشتیبانی نمی‌کند.  
> مثال زیر در کاساندرا **نامعتبر** است:
>
> ```sql
> INSERT INTO users (id, name) VALUES
>   (uuid(), 'Ali'),
>   (uuid(), 'Sara');
> ```
> هر `INSERT` باید فقط **یک ردیف** را درج کند. برای گروهی کردن، از `BEGIN BATCH` استفاده کنید.

---

## 🔍 کوئری SELECT در کاساندرا

در این بخش، نحوه **کوئری گرفتن از جدول** در کاساندرا با استفاده از CQL را بررسی می‌کنیم. همچنین بهترین شیوه‌ها، محدودیت‌ها و الگوهای کوئری را مرور خواهیم کرد.

---

### 📌 SELECT ساده

```sql
SELECT * FROM users;
```

تمام ستون‌ها و ردیف‌ها را از جدول `users` دریافت می‌کند.

```sql
SELECT id, name, email FROM users;
```

فقط ستون‌های مشخص‌شده را دریافت می‌کند.

---

### 🎯 کوئری با کلید پارتیشن

```sql
SELECT * FROM users WHERE id = some_uuid;
```

این کارآمدترین روش کوئری در کاساندراست — با **کلید پارتیشن**.  
کاساندرا مکان داده را دقیقاً می‌داند و مستقیماً از نود صحیح بازیابی می‌کند.

---

### ❌ کوئری بدون کلید پارتیشن (روش بد)

```sql
SELECT * FROM users WHERE name = 'Ali';
```

😬 این کوئری **از کلید پارتیشن استفاده نمی‌کند**، پس کاساندرا باید **تمام نودها را اسکن کند**.

> ⚠️ کاساندرا از معماری توزیع‌شده با **Token Ring** و **Partitioner + Token Map** استفاده می‌کند که شبیه جدول هش است. بدون کلید پارتیشن، نمی‌تواند از این بهینه‌سازی استفاده کند.

---

### 🔁 کوئری با `IN`

```sql
SELECT * FROM users WHERE id IN (uuid1, uuid2, uuid3);
```

امکان دریافت چند پارتیشن شناخته‌شده به‌صورت هم‌زمان را می‌دهد.  
> ✅ برای لیست‌های کوچک مناسب است — برای اسکن‌های گسترده توصیه نمی‌شود.

---

### 📉 LIMIT

```sql
SELECT * FROM users LIMIT 10;
```

فقط ۱۰ ردیف اول را دریافت می‌کند — برای صفحه‌بندی مفید است، ولی مرتب‌سازی سراسری را تضمین نمی‌کند.

---

### 🔃 ORDER BY (فقط درون یک پارتیشن)

```sql
SELECT * FROM users WHERE id = some_partition_key ORDER BY age DESC;
```

- `ORDER BY` فقط **در یک پارتیشن** کار می‌کند.
- ستون مرتب‌سازی باید **ستون خوشه‌بندی (clustering column)** باشد.

> ❗ استفاده از `ORDER BY` بدون فیلتر بر اساس کلید کامل پارتیشن منجر به خطا می‌شود.

---

### 🧪 ALLOW FILTERING

```sql
SELECT * FROM users WHERE age > 25 ALLOW FILTERING;
```

- امکان فیلتر بر اساس **ستون‌های غیر کلیدی** را فراهم می‌کند.
- ⚠️ ممکن است باعث **اسکن کل دیتاست** شود و **کارایی ندارد**.

> 💡 فقط زمانی از `ALLOW FILTERING` استفاده کنید که **داده کم باشد** یا **عملکرد قابل قبول باشد**.

---

### 📦 دسترسی به عناصر collection

**از `LIST`:**

```sql
SELECT interests[0] FROM users WHERE id = some_uuid;
```

اولین علاقه از لیست `interests` را بازمی‌گرداند.

**از `MAP`:**

```sql
SELECT settings['theme'] FROM users WHERE id = some_uuid;
SELECT keys(settings) FROM users WHERE id = some_uuid;
SELECT values(settings) FROM users WHERE id = some_uuid;
```

- دسترسی به مقادیر خاص
- دریافت تمام کلیدها یا مقادیر یک map

---

## ✏️ دستورات UPDATE و DELETE در کاساندرا

این راهنما نحوه بروزرسانی و حذف داده‌ها در کاساندرا با استفاده از CQL را توضیح می‌دهد. گرچه سینتکس آن مشابه SQL است، اما محدودیت‌ها و بهترین شیوه‌هایی وجود دارد.

---

### 🛠️ بروزرسانی (UPDATE)

```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE partition_key = value
  [AND clustering_column = value ...];
```

> ✅ باید **کلید پارتیشن** را در `WHERE` مشخص کنید.  
> ✅ اگر جدول از **ستون خوشه‌بندی** استفاده می‌کند، باید مقدار آن را هم تعیین کنید.

---

### 📦 مثال: جدول سفارش‌ها

```sql
CREATE TABLE orders (
    user_id UUID,
    order_date TIMESTAMP,
    status TEXT,
    total DECIMAL,
    PRIMARY KEY ((user_id), order_date)
);
```

---

#### ✅ بروزرسانی سفارش خاص:

```sql
UPDATE orders
SET status = 'shipped'
WHERE user_id = some_uuid_value AND order_date = some_timestamp_value;
```

---

### ⚠️ محدودیت‌های UPDATE

- ❌ نمی‌توان بدون تعیین **کلید پارتیشن**، ردیفی را بروزرسانی کرد.
- ❌ نمی‌توان مقدار **کلید پارتیشن** را تغییر داد.
- ❗ برای بروزرسانی یک ردیف خاص در جداول دارای ستون خوشه‌بندی، باید **کلید خوشه‌بندی کامل** را مشخص کنید.

---

### ❌ مثال نامعتبر UPDATE

```sql
UPDATE orders SET status = 'pending' WHERE status = 'shipped';
```

نامعتبر است — `status` کلید اولیه نیست.

---

### 🗑️ حذف (DELETE)

```sql
DELETE FROM table_name
WHERE partition_key = value
  [AND clustering_column = value ...];
```

اگر ستون خوشه‌بندی مشخص شود، یک ردیف خاص را حذف می‌کند. در غیر این صورت، کل پارتیشن را حذف می‌کند.

---

#### ✅ حذف یک سفارش خاص:

```sql
DELETE FROM orders
WHERE user_id = some_uuid_value AND order_date = some_timestamp_value;
```

#### 🧹 حذف یک پارتیشن کامل:

```sql
DELETE FROM orders
WHERE user_id = some_uuid_value;
```

> ⚠️ این کار تمام ردیف‌های دارای `user_id` مذکور را حذف می‌کند.

---

### 🧩 حذف آیتم از collection

```sql
UPDATE users SET interests = interests - ['music'] WHERE id = some_uuid_value;
```

`'music'` را از لیست `interests` برای کاربر مشخص‌شده حذف می‌کند.

---

> 📌 طراحی کوئری‌های خود را بر اساس نحوه توزیع و ذخیره داده‌ها در کاساندرا انجام دهید — از کلید پارتیشن برای کوئری‌گیری بهره ببرید و از فیلترهای گسترده بپرهیزید.


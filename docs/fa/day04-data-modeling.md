# 📚 روز چهارم: راهنمای مدل‌سازی داده در کاساندرا — گام‌به‌گام با مثال‌های عملی 🛠️
![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white) 
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)

به **روز چهارم** از سفر یادگیری آپاچی کاساندرا خوش آمدید! 🎉 در این راهنما، ما عمیقاً به موضوع **مدل‌سازی داده** می‌پردازیم، مهارتی کلیدی برای ساخت برنامه‌های مقیاس‌پذیر و کارآمد با کاساندرا. این چهارمین بخش از یک مجموعه آموزشی چهارروزه است:



تاکنون، شما با مفاهیم پایه کاساندرا آشنا شده‌اید، یک خوشه راه‌اندازی کرده‌اید و با CQL کار کرده‌اید. امروز، ما بر **مدل‌سازی داده مبتنی بر پرس‌وجو** تمرکز می‌کنیم و جدول‌هایی را طراحی می‌کنیم که برنامه‌های سریع و مقیاس‌پذیر را پشتیبانی کنند. بیایید مدل‌سازی داده را با مثال‌های واقعی سرگرم‌کننده و کاربردی کنیم! 🚀

> **نکته**: برای کاوش در تنظیمات کاساندرای خود، از این دستورات CQL استفاده کنید:
>
> - `DESCRIBE KEYSPACES;` — تمام فضای کلیدها (keyspaces) در خوشه را لیست می‌کند.
> - `DESCRIBE TABLES;` — تمام جداول در فضای کلید فعلی را لیست می‌کند. این دستورات را در پوسته CQL اجرا کنید تا با محیط خود آشنا شوید! 🔍

---

## 📖 مقدمه‌ای بر مدل‌سازی داده در کاساندرا

آپاچی کاساندرا یک پایگاه داده NoSQL توزیع‌شده است که برای **مقیاس‌پذیری بالا** و **دسترس‌پذیری** طراحی شده است. برخلاف پایگاه‌های داده SQL که به نرمال‌سازی و اتصال (JOIN) وابسته‌اند، کاساندرا از رویکرد **طراحی مبتنی بر پرس‌وجو** استفاده می‌کند. این یعنی شما جداول را بر اساس پرس‌وجوهای برنامه خود طراحی می‌کنید و اغلب داده‌ها را تکرار می‌کنید تا خواندن سریع تضمین شود. 🏎️

در این راهنما، ما:

- مفاهیم اصلی کاساندرا (کلیدهای اصلی، غیرنرمال‌سازی و غیره) را توضیح می‌دهیم 📝
- فرآیند مدل‌سازی داده را گام‌به‌گام مرور می‌کنیم 🗺️
- مثال‌های هیجان‌انگیزی می‌سازیم: یک **برنامه شبکه اجتماعی** 📱 و یک **کتاب‌فروشی آنلاین** 📚
- بهترین روش‌ها و تله‌های رایج را به اشتراک می‌گذاریم ⚠️
- تمرین‌هایی برای آزمایش مهارت‌های شما ارائه می‌دهیم 🧠

بیایید شروع کنیم! 💪

---

## 🔑 مفاهیم اصلی مدل‌سازی داده در کاساندرا

برای طراحی جداول کارآمد در کاساندرا، باید این مفاهیم بنیادی را درک کنید:

### 1. کلید اصلی 🗝️

**کلید اصلی** به‌طور منحصربه‌فرد ردیف‌ها را شناسایی می‌کند و شامل موارد زیر است:

- **کلید پارتیشن**: تعیین می‌کند که داده در کدام گره خوشه ذخیره شود. این کلید برای توزیع یکنواخت داده‌ها هش می‌شود. برای مثال، شناسه کاربر داده‌های هر کاربر را کنار هم نگه می‌دارد. 📍
- **ستون‌های خوشه‌بندی** (اختیاری): داده‌ها را درون یک پارتیشن مرتب می‌کنند. برای مثال، مرتب‌سازی پست‌ها بر اساس زمان در پارتیشن یک کاربر. 📅

مثال:

```sql
PRIMARY KEY ((partition_key), clustering_column1, clustering_column2)
```

- `partition_key` گره ذخیره‌سازی را تعیین می‌کند.
- `clustering_column1` و `clustering_column2` ردیف‌ها را درون پارتیشن مرتب می‌کنند.

### 2. غیرنرمال‌سازی 📚

در SQL، داده‌ها را نرمال می‌کنید تا از تکرار جلوگیری شود. در کاساندرا، شما **غیرنرمال‌سازی** می‌کنید و جداول متعددی متناسب با پرس‌وجوهای خاص ایجاد می‌کنید. این یعنی داده‌ها را تکرار می‌کنید تا نیازی به JOIN نداشته باشید. مثل این است که قفسه‌های کتاب را برای ژانرهای مورد علاقه هر خواننده از قبل آماده کنید! 📚

### 3. عدم پشتیبانی از JOIN و تجمیع 🚫

کاساندرا از JOIN، GROUP BY یا تجمیع‌های آنی (مانند SUM، COUNT) پشتیبانی نمی‌کند. تمام داده‌های مورد نیاز برای یک پرس‌وجو باید در یک جدول باشند. برای تجمیع، از **ستون‌های شمارشگر** یا نتایج از پیش محاسبه‌شده استفاده کنید. مثل پختن نتایج پرس‌وجو از قبل! 🍰

### 4. توزیع داده 🌐

کلید پارتیشن داده‌ها را بین گره‌ها هش می‌کند. یک کلید پارتیشن خوب (مثل شناسه کاربر یا تاریخ) توزیع یکنواخت را تضمین می‌کند و از "نقاط داغ" (جایی که یک گره بیش از حد بارگذاری شود) جلوگیری می‌کند. تصور کنید کتاب‌ها را به‌طور یکنواخت بین شعب کتابخانه پخش کنید! 🏛️

---

## 📋 سینتکس جدول در کاساندرا

سینتکس پایه برای ایجاد جدول در کاساندرا:

```sql
CREATE TABLE keyspace_name.table_name (
    column1 data_type,
    column2 data_type,
    PRIMARY KEY ((partition_key), clustering_column1)
) WITH CLUSTERING ORDER BY (clustering_column1 DESC);
```

- **فضای کلید (Keyspace)**: فضایی برای جداول، مشابه یک پایگاه داده در SQL.
- **کلید پارتیشن**: در پرانتز برای توزیع داده.
- **ستون‌های خوشه‌بندی**: داده‌ها را درون پارتیشن مرتب می‌کنند.
- **CLUSTERING ORDER BY**: ترتیب صعودی (`ASC`) یا نزولی (`DESC`) را تعیین می‌کند.

مثال:

```sql
CREATE TABLE bookstore.books_by_genre (
    genre text,
    published_date timestamp,
    book_id uuid,
    title text,
    author text,
    PRIMARY KEY (genre, published_date)
) WITH CLUSTERING ORDER BY (published_date DESC);
```

- **کلید پارتیشن**: `genre` (کتاب‌ها را بر اساس ژانر گروه‌بندی می‌کند، مثلاً "فانتزی").
- **ستون خوشه‌بندی**: `published_date` (کتاب‌ها را در هر ژانر جدید به قدیم مرتب می‌کند).

پرس‌وجو:

```sql
SELECT * FROM books_by_genre WHERE genre = 'Fantasy';
```

---

## 🗺️ فرآیند مدل‌سازی داده گام‌به‌گام

برای ایجاد یک مدل داده در کاساندرا:

1. **شناسایی پرس‌وجوها** 📋: تمام پرس‌وجوهای مورد نیاز برنامه را فهرست کنید (مثلاً "گرفتن همه کتاب‌ها بر اساس ژانر").
2. **تعریف موجودیت‌ها** 🧩: موجودیت‌ها (مثل کتاب، کاربر) و روابط آن‌ها را شناسایی کنید.
3. **طراحی جداول** 🏗️: برای هر پرس‌وجو یک جدول بسازید که تمام داده‌های مورد نیاز را شامل شود.
4. **انتخاب کلیدها** 🔑: کلیدهای پارتیشن و خوشه‌بندی را برای توزیع و مرتب‌سازی انتخاب کنید.
5. **اعتبارسنجی** ✅: با داده‌های واقعی آزمایش کنید تا عملکرد را تضمین کنید.

بیایید این را با دو مثال جذاب به کار ببریم! 🎉

---

## 📱 مثال عملی 1: برنامه شبکه اجتماعی

تصور کنید در حال ساخت **ConnectSphere** هستید، یک برنامه شبکه اجتماعی که کاربران در آن عکس پست می‌کنند، نظر می‌دهند و لایک می‌کنند. بیایید داده‌های آن را مدل کنیم.

### موجودیت‌ها

- **کاربران**: شناسه، نام کاربری، ایمیل، تاریخ پیوستن.
- **پست‌ها**: شناسه، شناسه کاربر، کپشن، آدرس تصویر، زمان ایجاد.
- **نظرات**: شناسه، شناسه پست، شناسه کاربر، متن، زمان نظر.
- **لایک‌ها**: شناسه کاربر، شناسه پست، زمان لایک.

### نیازهای پرس‌وجو

1. گرفتن پروفایل کاربر با شناسه. 🧑
2. گرفتن پست‌های یک کاربر، مرتب‌شده بر اساس زمان (جدید به قدیم). 📸
3. گرفتن یک پست خاص با شناسه. 📷
4. گرفتن نظرات یک پست، مرتب‌شده بر اساس زمان (قدیم به جدید). 💬
5. گرفتن پست‌هایی که کاربر لایک کرده. ❤️
6. گرفتن کاربرانی که یک پست را لایک کرده‌اند. 👍

### جداول

#### 1. کاربران

```sql
CREATE TABLE connectsphere.users (
    user_id uuid PRIMARY KEY,
    username text,
    email text,
    joined_at timestamp
);
```

- **هدف**: گرفتن جزئیات کاربر (مثلاً "پروفایل @TravelLover را نشان بده").
- **پرس‌وجو**: `SELECT * FROM users WHERE user_id = ?;`

#### 2. پست‌ها بر اساس کاربر

```sql
CREATE TABLE connectsphere.posts_by_user (
    user_id uuid,
    created_at timestamp,
    post_id uuid,
    caption text,
    image_url text,
    PRIMARY KEY ((user_id), created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);
```

- **هدف**: نمایش پست‌های کاربر، مثل فید عکس @TravelLover، جدید به قدیم.
- **پرس‌وجو**: `SELECT * FROM posts_by_user WHERE user_id = ?;`

#### 3. پست بر اساس شناسه

```sql
CREATE TABLE connectsphere.post_by_id (
    post_id uuid PRIMARY KEY,
    user_id uuid,
    caption text,
    image_url text,
    created_at timestamp
);
```

- **هدف**: گرفتن یک پست خاص، مثل یک عکس غروب پرطرفدار. 🌅
- **پرس‌وجو**: `SELECT * FROM post_by_id WHERE post_id = ?;`

#### 4. نظرات بر اساس پست

```sql
CREATE TABLE connectsphere.comments_by_post (
    post_id uuid,
    comment_time timestamp,
    comment_id uuid,
    user_id uuid,
    content text,
    PRIMARY KEY ((post_id), comment_time)
) WITH CLUSTERING ORDER BY (comment_time ASC);
```

- **هدف**: نمایش نظرات یک پست، مثل واکنش‌ها به یک عکس سفر.
- **پرس‌وجو**: `SELECT * FROM comments_by_post WHERE post_id = ?;`

#### 5. لایک‌ها بر اساس کاربر

```sql
CREATE TABLE connectsphere.likes_by_user (
    user_id uuid,
    liked_at timestamp,
    post_id uuid,
    PRIMARY KEY ((user_id), liked_at)
) WITH CLUSTERING ORDER BY (liked_at DESC);
```

- **هدف**: نمایش پست‌هایی که کاربر لایک کرده، مثل عکس‌های مورد علاقه @TravelLover.
- **پرس‌وجو**: `SELECT * FROM likes_by_user WHERE user_id = ?;`

#### 6. لایک‌ها بر اساس پست

```sql
CREATE TABLE connectsphere.likes_by_post (
    post_id uuid,
    liked_at timestamp,
    user_id uuid,
    PRIMARY KEY ((post_id), liked_at)
) WITH CLUSTERING ORDER BY (liked_at DESC);
```

- **هدف**: نمایش کاربرانی که یک پست را لایک کرده‌اند، مثل طرفداران یک عکس غروب. 🌄
- **پرس‌وجو**: `SELECT * FROM likes_by_post WHERE post_id = ?;`

### نمونه پرس‌وجوها

```sql
-- گرفتن پروفایل @TravelLover
SELECT * FROM users WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- گرفتن پست‌های @TravelLover
SELECT * FROM posts_by_user WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- گرفتن یک عکس غروب خاص
SELECT * FROM post_by_id WHERE post_id = 660e8400-e29b-41d4-a716-446655440000;

-- گرفتن نظرات یک پست
SELECT * FROM comments_by_post WHERE post_id = 660e8400-e29b-41d4-a716-446655440000;

-- گرفتن پست‌هایی که @TravelLover لایک کرده
SELECT * FROM likes_by_user WHERE user_id = 550e8400-e29b-41d4-a716-446655440000;

-- گرفتن کاربرانی که یک پست را لایک کرده‌اند
SELECT * FROM likes_by_post WHERE post_id = 660e8400-e29b-41d4-a716-446655440000;
```

---

## 📚 مثال عملی 2: کتاب‌فروشی آنلاین

بیایید **BookHaven** را بسازیم، یک کتاب‌فروشی آنلاین که کاربران در آن کتاب می‌خرند و مدیران فروش را رصد می‌کنند. 📖

### موجودیت‌ها

- **کاربران**: شناسه، نام، ایمیل، تلفن، تاریخ پیوستن.
- **کتاب‌ها**: شناسه، عنوان، نویسنده، ژانر، تاریخ انتشار.
- **سفارش‌ها**: شناسه، شناسه کاربر، زمان سفارش، قیمت کل، کتاب‌ها، وضعیت.
- **خلاصه فروش ماهانه**: کل فروش، تعداد سفارش‌ها بر اساس ماه.

### نیازهای پرس‌وجو

1. گرفتن کاربر با شناسه. 🧑
2. گرفتن کتاب‌ها بر اساس ژانر، مرتب‌شده بر اساس تاریخ انتشار. 📚
3. گرفتن سفارش‌های کاربر، مرتب‌شده بر اساس زمان. 📦
4. گرفتن سفارش‌های یک روز خاص. 📅
5. گرفتن خلاصه فروش ماهانه. 💰

### جداول

#### 1. کاربران

```sql
CREATE TABLE bookhaven.users (
    user_id uuid PRIMARY KEY,
    name text,
    email text,
    phone text,
    joined_at timestamp
);
```

- **هدف**: گرفتن جزئیات کاربر، مثل "آلیس کتاب‌دوست کیست؟".
- **پرس‌وجو**: `SELECT * FROM users WHERE user_id = ?;`

#### 2. کتاب‌ها بر اساس ژانر

```sql
CREATE TABLE bookhaven.books_by_genre (
    genre text,
    published_date timestamp,
    book_id uuid,
    title text,
    author text,
    price decimal,
    PRIMARY KEY ((genre), published_date)
) WITH CLUSTERING ORDER BY (published_date DESC);
```

- **هدف**: مرور کتاب‌ها بر اساس ژانر، مثل "جدیدترین رمان‌های فانتزی".
- **پرس‌وجو**: `SELECT * FROM books_by_genre WHERE genre = 'Fantasy';`

#### 3. سفارش‌ها بر اساس کاربر

```sql
CREATE TABLE bookhaven.orders_by_user (
    user_id uuid,
    order_time timestamp,
    order_id uuid,
    total_price decimal,
    books list<text>,
    status text,
    PRIMARY KEY ((user_id), order_time)
) WITH CLUSTERING ORDER BY (order_time DESC);
```

- **هدف**: نمایش تاریخچه سفارش‌های کاربر، مثل خریدهای کتاب آلیس.
- **پرس‌وجو**: `SELECT * FROM orders_by_user WHERE user_id = ?;`

#### 4. سفارش‌ها بر اساس روز

```sql
CREATE TABLE bookhaven.orders_by_day (
    order_date date,
    order_time timestamp,
    order_id uuid,
    user_id uuid,
    total_price decimal,
    books list<text>,
    status text,
    PRIMARY KEY ((order_date), order_time)
) WITH CLUSTERING ORDER BY (order_time DESC);
```

- **هدف**: مشاهده تمام سفارش‌های یک روز خاص، مثل فروش‌های جمعه سیاه.
- **پرس‌وجو**: `SELECT * FROM orders_by_day WHERE order_date = '2025-08-01';`

#### 5. خلاصه فروش ماهانه

```sql
CREATE TABLE bookhaven.monthly_sales_summary (
    month text PRIMARY KEY,
    total_sales counter,
    order_count counter
);
```

- **هدف**: ردیابی فروش ماهانه، مثل درآمد ماه اوت.
- **پرس‌وجو**: `SELECT * FROM monthly_sales_summary WHERE month = '2025-08';`

### درج‌ها

```sql
-- افزودن کاربر آلیس کتاب‌دوست
INSERT INTO users (user_id, name, email, phone, joined_at) VALUES (
    550e8400-e29b-41d4-a716-446655440000,
    'آلیس کتاب‌دوست',
    'alice@example.com',
    '09123456789',
    toTimestamp(now())
);

-- افزودن یک کتاب فانتزی
INSERT INTO books_by_genre (
    genre, published_date, book_id, title, author, price
) VALUES (
    'Fantasy',
    '2025-01-01 00:00:00',
    770e8400-e29b-41d4-a716-446655440000,
    'جستجوی اژدها',
    'جی.آر.آر. تالکین',
    29.99
);

-- افزودن سفارش آلیس
INSERT INTO orders_by_user (
    user_id, order_time, order_id, total_price, books, status
) VALUES (
    550e8400-e29b-41d4-a716-446655440000,
    toTimestamp(now()),
    660e8400-e29b-41d4-a716-446655440000,
    29.99,
    ['جستجوی اژدها'],
    'تحویل‌شده'
);

-- افزودن سفارش به جدول روزانه
INSERT INTO orders_by_day (
    order_date, order_time, order_id, user_id, total_price, books, status
) VALUES (
    '2025-08-01',
    toTimestamp(now()),
    660e8400-e29b-41d4-a716-446655440000,
    550e8400-e29b-41d4-a716-446655440000,
    29.99,
    ['جستجوی اژدها'],
    'تحویل‌شده'
);

-- به‌روزرسانی خلاصه فروش
UPDATE monthly_sales_summary SET 
    total_sales = total_sales + 29.99,
    order_count = order_count + 1 
WHERE month = '2025-08';
```

---

## ✅ بهترین روش‌ها

- **طراحی مبتنی بر پرس‌وجو** 📋: جداول را برای پرس‌وجوهای خاص بسازید.
- **پارتیشن‌های متعادل** ⚖️: کلیدهایی انتخاب کنید که داده را به‌طور یکنواخت توزیع کنند.
- **پارتیشن‌های کوچک** 📏: پارتیشن‌ها را زیر چند هزار ردیف نگه دارید.
- **غیرنرمال‌سازی هوشمند** 📚: داده را به‌طور استراتژیک تکرار کنید.
- **تجمیع از پیش محاسبه‌شده** 📊: از شمارشگرها یا جداول برای تجمیع استفاده کنید.
- **آزمایش کامل** 🧪: با داده‌های واقعی اعتبارسنجی کنید.

---

## ⚠️ تله‌های رایج

- **تفکر SQL** 🚫: نرمال‌سازی نکنید یا انتظار JOIN نداشته باشید.
- **نقاط داغ** 🔥: از کلیدهایی که گره‌ها را بیش از حد بارگذاری می‌کنند اجتناب کنید.
- **پارتیشن‌های بزرگ** 📈: از کلیدهای ترکیبی برای تقسیم داده استفاده کنید.
- **استفاده بیش از حد از ایندکس‌ها** 📉: به جای ایندکس‌های ثانویه، جدول بسازید.
- **به‌روزرسانی‌های مکرر** 🔄: برای بارهای کاری append-heavy طراحی کنید.

---

## 🧠 تمرین‌ها

1. **کتاب‌ها بر اساس نویسنده** 📖

   - پرس‌وجو: `SELECT * FROM books_by_author WHERE author = ?;`
   - راهنمایی: از `author` به‌عنوان کلید پارتیشن و `published_date` به‌عنوان ستون خوشه‌بندی استفاده کنید.

2. **نقدهای کاربران** ⭐

   - پرس‌وجوها: گرفتن نقدها بر اساس کتاب یا کاربر.
   - راهنمایی: جداول `reviews_by_book` و `reviews_by_user` بسازید.

3. **به‌روزرسانی وضعیت سفارش** 📦

   - پرس‌وجو: ردیابی تغییرات وضعیت سفارش با شناسه سفارش.
   - راهنمایی: از `order_id` به‌عنوان کلید پارتیشن و `update_time` به‌عنوان ستون خوشه‌بندی استفاده کنید.

---

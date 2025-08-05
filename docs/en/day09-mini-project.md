# روز نهم - مینی پروژه: ساخت اپلیکیشن کتابخانه ساده با آپاچی کاساندرا

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)

به **روز نهم** از سفر یادگیری آپاچی کاساندرا خوش آمدید! 🎉 امروز یک **مینی پروژه عملی** را انجام می‌دهیم که در آن یک اپلیکیشن بک‌اند ساده برای مدیریت یک کتابخانه دیجیتال طراحی و پیاده‌سازی می‌کنیم. این پروژه با استفاده از رویکرد **Query-Driven** و ویژگی‌های پیشرفته CQL مانند مجموعه‌ها (SET, LIST, MAP)، TTL، و ایندکس‌های ثانویه ساخته می‌شود. هدف این است که شما بتوانید دانش خود را در یک سناریوی واقعی به کار ببرید و برای توسعه API یا رابط کاربری در آینده آماده شوید. 🚀

در این راهنما، ما:

- جداول را بر اساس نیازهای کوئری طراحی می‌کنیم 📝
- از مجموعه‌های SET، LIST، و MAP برای مدل‌سازی داده‌های پیچیده استفاده می‌کنیم 📚
- از TTL برای حذف خودکار داده‌های منقضی‌شده بهره می‌بریم 🕒
- ایندکس‌های ثانویه را برای جستجوی سریع‌تر پیاده‌سازی می‌کنیم 🔍
- کوئری‌های کاربردی و تحلیل داده‌ها را بررسی می‌کنیم 📊
- تمرین‌هایی برای تسلط بیشتر ارائه می‌دهیم 💪

---

## 🎯 هدف پروژه

ساخت یک اپلیکیشن بک‌اند ساده برای مدیریت یک کتابخانه دیجیتال با تمرکز بر اصول طراحی داده در کاساندرا و استفاده از ویژگی‌های پیشرفته CQL. این پروژه به شما کمک می‌کند تا:

- جداول را بر اساس نیازهای کوئری طراحی کنید.
- از مجموعه‌های داده‌ای و TTL برای مدیریت کارآمد داده‌ها استفاده کنید.
- ایندکس‌های ثانویه را برای بهبود جستجو پیاده‌سازی کنید.
- برای مقیاس‌پذیری و توسعه‌های آینده آماده شوید.

## 📖 سناریو پروژه

فرض کنید یک کتابخانه دیجیتال داریم که ویژگی‌های زیر را ارائه می‌دهد:

- **ثبت کاربر**: ذخیره اطلاعات کاربران شامل نام، ایمیل، تاریخ ثبت‌نام، و علاقه‌مندی‌ها (به‌صورت MAP).
- **ثبت کتاب**: ذخیره اطلاعات کتاب‌ها شامل عنوان، نویسنده، دسته‌بندی‌ها (SET)، سال انتشار، و وضعیت موجودی.
- **امانت‌گیری**: امکان امانت گرفتن چند کتاب توسط هر کاربر و ثبت تاریخچه امانت‌ها.
- **تاریخچه**: نگهداری تاریخچه امانت‌ها برای هر کاربر و هر کتاب.
- **گزارش‌گیری**: گزارش تعداد کتاب‌های فعال، کتاب‌های بازنگشته، و کاربران فعال.
- **خودکارسازی**: حذف خودکار داده‌های قدیمی (مثل امانت‌های قدیمی) با استفاده از TTL.

---

## 🧱 گام ۱: طراحی دیتابیس (Query-Driven)

در کاساندرا، جداول بر اساس کوئری‌هایی که اپلیکیشن اجرا خواهد کرد طراحی می‌شوند، نه صرفاً بر اساس نرمال‌سازی سنتی. در ادامه، جداول موردنیاز را برای پاسخگویی به کوئری‌های پروژه طراحی می‌کنیم.

### 🗃️ جدول ۱: users – ثبت کاربران

این جدول اطلاعات کاربران را ذخیره می‌کند، با استفاده از MAP برای ذخیره علاقه‌مندی‌ها به دسته‌بندی‌های کتاب.

```sql
CREATE TABLE library.users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    registration_date TIMESTAMP,
    preferences MAP<TEXT, INT>  -- علاقه‌مندی‌ها به دسته‌بندی‌ها با امتیاز
);
```

### 📚 جدول ۲: books – ثبت کتاب‌ها

این جدول اطلاعات کتاب‌ها را ذخیره می‌کند، با استفاده از SET برای دسته‌بندی‌ها.

```sql
CREATE TABLE library.books (
    book_id UUID PRIMARY KEY,
    title TEXT,
    author TEXT,
    categories SET<TEXT>,
    published_year INT,
    available BOOLEAN
);
```

### 📦 جدول ۳: borrows_by_user – امانت‌های هر کاربر

این جدول تاریخچه امانت‌های هر کاربر را ذخیره می‌کند و از TTL برای حذف خودکار رکوردها پس از ۳۰ روز استفاده می‌کند.

```sql
CREATE TABLE library.borrows_by_user (
    user_id UUID,
    borrow_id TIMEUUID,
    book_id UUID,
    borrow_date TIMESTAMP,
    return_date TIMESTAMP,
    notes TEXT,
    PRIMARY KEY (user_id, borrow_id)
) WITH CLUSTERING ORDER BY (borrow_id DESC)
  AND default_time_to_live = 2592000;  -- حذف خودکار پس از ۳۰ روز
```

### 📦 جدول ۴: borrows_by_book – تاریخچه امانت هر کتاب

این جدول تاریخچه امانت‌ها را از منظر کتاب‌ها ذخیره می‌کند.

```sql
CREATE TABLE library.borrows_by_book (
    book_id UUID,
    borrow_id TIMEUUID,
    user_id UUID,
    borrow_date TIMESTAMP,
    return_date TIMESTAMP,
    status TEXT,
    PRIMARY KEY (book_id, borrow_id)
) WITH CLUSTERING ORDER BY (borrow_id DESC);
```

### 🔍 ایندکس ثانویه: جستجو بر اساس عنوان کتاب

برای پشتیبانی از جستجوی کتاب‌ها بر اساس عنوان، یک ایندکس ثانویه ایجاد می‌کنیم.

```sql
CREATE INDEX books_title_idx ON library.books (title);
```

---

## 🧪 گام ۲: درج داده‌های اولیه

برای آزمایش سیستم، داده‌های اولیه برای کاربران و کتاب‌ها درج می‌کنیم.

### ➕ درج کاربران

```sql
INSERT INTO library.users (user_id, name, email, registration_date, preferences)
VALUES (uuid(), 'علی رضایی', 'ali@example.com', toTimestamp(now()), {'Fantasy': 5, 'Science': 3});

INSERT INTO library.users (user_id, name, email, registration_date, preferences)
VALUES (uuid(), 'سارا احمدی', 'sara@example.com', toTimestamp(now()), {'Classic': 4, 'History': 2});
```

### ➕ درج کتاب‌ها

```sql
INSERT INTO library.books (book_id, title, author, categories, published_year, available)
VALUES (uuid(), '1984', 'جورج اورول', {'دیستوپیایی', 'کلاسیک'}, 1949, true);

INSERT INTO library.books (book_id, title, author, categories, published_year, available)
VALUES (uuid(), 'هابیت', 'جی.آر.آر. تالکین', {'فانتزی'}, 1937, true);

INSERT INTO library.books (book_id, title, author, categories, published_year, available)
VALUES (uuid(), 'انسان خردمند', 'یووال نوح هراری', {'تاریخ', 'غیرداستانی'}, 2011, true);
```

---

## 📥 گام ۳: ثبت امانت کتاب

برای ثبت امانت، باید اطلاعات را در هر دو جدول `borrows_by_user` و `borrows_by_book` ذخیره کنیم و وضعیت موجودی کتاب را به‌روزرسانی کنیم.

### ⛏️ مراحل:

1. دریافت `user_id` و `book_id`.
2. تولید `borrow_id` با استفاده از `now()` (نوع TIMEUUID).
3. ثبت در `borrows_by_user`.
4. ثبت در `borrows_by_book`.
5. به‌روزرسانی وضعیت کتاب در جدول `books`.

### ✅ عملیات ثبت امانت:

```sql
-- ثبت در borrows_by_user
INSERT INTO library.borrows_by_user (user_id, borrow_id, book_id, borrow_date, return_date, notes)
VALUES (
  <user_id>,
  now(),
  <book_id>,
  toTimestamp(now()),
  null,
  'اولین امانت'
);

-- ثبت در borrows_by_book
INSERT INTO library.borrows_by_book (book_id, borrow_id, user_id, borrow_date, return_date, status)
VALUES (
  <book_id>,
  now(),
  <user_id>,
  toTimestamp(now()),
  null,
  'borrowed'
);

-- به‌روزرسانی وضعیت کتاب
UPDATE library.books SET available = false WHERE book_id = <book_id>;
```

---

## 📤 گام ۴: بازگرداندن کتاب

برای ثبت بازگشت کتاب، باید جداول امانت و وضعیت کتاب را به‌روزرسانی کنیم.

### ✅ عملیات بازگشت:

```sql
-- به‌روزرسانی در borrows_by_user
UPDATE library.borrows_by_user 
SET return_date = toTimestamp(now()), notes = 'کتاب بازگشت داده شد'
WHERE user_id = <user_id> AND borrow_id = <borrow_id>;

-- به‌روزرسانی در borrows_by_book
UPDATE library.borrows_by_book 
SET return_date = toTimestamp(now()), status = 'returned'
WHERE book_id = <book_id> AND borrow_id = <borrow_id>;

-- به‌روزرسانی موجودی کتاب
UPDATE library.books SET available = true WHERE book_id = <book_id>;
```

---

## 🔍 گام ۵: کوئری‌های کاربردی

این کوئری‌ها برای استخراج اطلاعات و گزارش‌گیری از داده‌ها استفاده می‌شوند.

### 🟢 ۱. لیست کتاب‌های در دسترس

```sql
SELECT * FROM library.books WHERE available = true ALLOW FILTERING;
```

> **هشدار**: استفاده از `ALLOW FILTERING` می‌تواند عملکرد را کاهش دهد. برای کوئری‌های پرتکرار، بهتر است از ایندکس یا جدول جداگانه استفاده کنید.

### 🔎 ۲. جستجوی کتاب بر اساس عنوان

```sql
SELECT * FROM library.books WHERE title = '1984';
```

### 👤 ۳. مشاهده امانت‌های یک کاربر خاص

```sql
SELECT * FROM library.borrows_by_user WHERE user_id = <user_id>;
```

### 📚 ۴. مشاهده تاریخچه امانت یک کتاب

```sql
SELECT * FROM library.borrows_by_book WHERE book_id = <book_id>;
```

### 📊 ۵. گزارش کتاب‌های امانت‌گرفته‌شده در ۷ روز گذشته

```sql
SELECT * FROM library.borrows_by_book 
WHERE borrow_date >= '2025-07-29T00:00:00' 
AND borrow_date <= toTimestamp(now()) 
ALLOW FILTERING;
```

---

## 🔄 گام ۶: پاک‌سازی خودکار با TTL

استفاده از `default_time_to_live = 2592000` (۳۰ روز) در جدول `borrows_by_user` باعث می‌شود رکوردهای امانت پس از ۳۰ روز به‌صورت خودکار حذف شوند.

### مزایا:

- **سبک ماندن جداول**: کاهش حجم داده‌های قدیمی.
- **مدیریت خودکار**: نیازی به اسکریپت‌های اضافی برای حذف داده‌ها نیست.
- **بهینه‌سازی حافظه**: استفاده کارآمد از منابع دیتابیس.

---

## 📝 تمرین‌ها برای تسلط کامل

برای تقویت مهارت‌های خود، این تمرین‌ها را انجام دهید:

### ✅ تمرین ۱: درج داده

برای ۵ کاربر مختلف و حداقل ۱۰ کتاب داده درج کنید. از دسته‌بندی‌های متنوع در SET و امتیازهای مختلف در MAP استفاده کنید.

### ✅ تمرین ۲: ثبت امانت

برای هر کاربر حداقل ۲ کتاب امانت ثبت کنید. مطمئن شوید که دسته‌بندی‌های کتاب‌ها با علاقه‌مندی‌های کاربران همخوانی دارد.

### ✅ تمرین ۳: جستجوی علاقه‌مندی‌ها

کوئری‌ای بنویسید که کاربران علاقه‌مند به دسته‌بندی "Fantasy" را پیدا کند. (نکته: از `CONTAINS KEY` برای MAP در جدول `users` استفاده کنید.)

```sql
SELECT * FROM library.users WHERE preferences CONTAINS KEY 'Fantasy';
```

### ✅ تمرین ۴: گزارش امانت‌های اخیر

کوئری‌ای بنویسید که تمام کتاب‌هایی که در ۷ روز گذشته امانت گرفته شده‌اند را برگرداند. (مثال در بالا ارائه شده است.)

### ✅ تمرین ۵: اسکریپت پایتون برای بررسی موجودی

یک اسکریپت پایتونی بنویسید که موجودی کتاب‌ها را بررسی کند و اگر هیچ نسخه‌ای در دسترس نباشد، هشداری نمایش دهد.

#### مثال اسکریپت پایتون:

```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

# تنظیمات اتصال به کاساندرا
auth_provider = PlainTextAuthProvider(username='cassandra', password='cassandra')
cluster = Cluster(['127.0.0.1'], auth_provider=auth_provider)
session = cluster.connect('library')

# بررسی کتاب‌های در دسترس
rows = session.execute("SELECT title, available FROM books")
for row in rows:
    if not row.available:
        print(f"هشدار: کتاب '{row.title}' در دسترس نیست!")

# بستن اتصال
cluster.shutdown()
```

---

## 🛠️ نکات اضافی و بهترین روش‌ها

### بهترین روش‌ها

- **طراحی کوئری‌محور** 📝: همیشه جداول را بر اساس کوئری‌های اپلیکیشن طراحی کنید.
- **استفاده از مجموعه‌ها** 🗂️: از SET، LIST، و MAP برای مدل‌سازی داده‌های پیچیده اما مرتبط استفاده کنید.
- **مدیریت TTL** 🕒: برای داده‌های موقتی (مثل امانت‌ها) از TTL استفاده کنید تا خوشه سبک بماند.
- **ایندکس‌های ثانویه با احتیاط** 🔍: ایندکس‌ها را فقط برای ستون‌های با کاردینالیتی پایین استفاده کنید.
- **تست کوئری‌ها** 🧪: قبل از استفاده در محیط تولید، کوئری‌ها را در محیط آزمایشی آزمایش کنید.

### اشتباهات رایج

- **استفاده بیش از حد از ALLOW FILTERING** ⚠️: این کار می‌تواند عملکرد را کاهش دهد. به جای آن، جداول را بازطراحی کنید.
- **نادیده گرفتن TTL** 🚫: عدم استفاده از TTL برای داده‌های موقتی می‌تواند خوشه را سنگین کند.
- **استفاده نادرست از ایندکس‌های ثانویه** 🚨: ایندکس‌ها برای ستون‌های با کاردینالیتی بالا (مثل ایمیل) مناسب نیستند.

---

## 🚀 نتیجه‌گیری

در این مینی پروژه، یک اپلیکیشن بک‌اند ساده برای مدیریت کتابخانه دیجیتال با استفاده از آپاچی کاساندرا طراحی کردیم. با استفاده از طراحی کوئری‌محور، مجموعه‌های SET، LIST، و MAP، TTL، و ایندکس‌های ثانویه، توانستیم سیستمی مقیاس‌پذیر و کارآمد ایجاد کنیم. این پروژه پایه‌ای محکم برای اضافه کردن API یا رابط کاربری در آینده فراهم می‌کند. با انجام تمرین‌های پیشنهادی، مهارت‌های خود را در CQL و کاساندرا تقویت کنید! 💪
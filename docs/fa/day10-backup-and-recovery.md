# روز دهم - استراتژی‌های بکاپ و ریکاوری در آپاچی کاساندرا

![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white)

به **روز دهم** از سفر یادگیری آپاچی کاساندرا خوش آمدید! 🎉 امروز به بررسی **استراتژی‌های بکاپ و ریکاوری** در کاساندرا می‌پردازیم، که یکی از مهم‌ترین جنبه‌های مدیریت یک سیستم توزیع‌شده است. این راهنما شامل روش‌های خودکار و دستی برای بازیابی داده‌ها، استفاده از Snapshotها، و دستورات خاص برای محیط‌های Docker است. با یادگیری این تکنیک‌ها، می‌توانید از داده‌های خود در برابر خرابی‌ها محافظت کرده و سیستم خود را پایدار نگه دارید. 🚀

در این راهنما، ما:

- روش‌های بازیابی خودکار و دستی در کاساندرا را بررسی می‌کنیم 📝
- نحوه استفاده از Commit Log و Snapshot برای بکاپ و ریکاوری را توضیح می‌دهیم 🛠️
- دستورات مربوط به محیط Docker را ارائه می‌دهیم 🐳
- ابزارهای مهم و نکات کلیدی برای استراتژی ریکاوری را مرور می‌کنیم 🔍
- یک مثال عملی برای بکاپ و ریکاوری ارائه می‌دهیم 📚

---

## 🎯 هدف این روز

هدف ما یادگیری روش‌های مختلف برای حفاظت و بازیابی داده‌ها در کاساندرا است، از جمله:

- استفاده از مکانیزم‌های داخلی مانند Hinted Handoff، Read Repair، و Commit Log برای بازیابی خودکار.
- ایجاد و مدیریت Snapshotها برای بکاپ‌گیری دستی.
- اجرای دستورات بکاپ و ریکاوری در محیط‌های Docker.
- پیاده‌سازی استراتژی‌های پایدار برای محیط‌های تولیدی.

---

## 🛠️ روش‌های بازیابی داده‌ها در کاساندرا

### ریکاوری خودکار (Automatic Recovery)

کاساندرا از مکانیزم‌های داخلی برای بازیابی داده‌ها در صورت خرابی نودها استفاده می‌کند:

- **Hinted Handoff**: زمانی که یک نود از دسترس خارج می‌شود، داده‌های ارسالی به آن نود به‌صورت موقت توسط نودهای دیگر ذخیره می‌شوند. پس از بازگشت نود، این داده‌ها به آن منتقل می‌شوند.
- **Read Repair**: هنگام خواندن داده‌ها، کاساندرا ناسازگاری‌های بین Replicaها را شناسایی و اصلاح می‌کند.
- **Anti-Entropy Repair**: با اجرای دستور `nodetool repair`، داده‌ها بین Replicaها همگام‌سازی می‌شوند تا از یکپارچگی داده‌ها اطمینان حاصل شود.

### بازیابی با Commit Log

اگر یک نود در حال اجرا کرش کند، کاساندرا هنگام راه‌اندازی مجدد، Commit Log را می‌خواند تا تغییراتی که هنوز به SSTable منتقل نشده‌اند را بازیابی کند. این فرآیند تضمین می‌کند که هیچ داده‌ای از دست نرود.

### بازیابی با بکاپ (Snapshot)

کاساندرا به‌صورت پیش‌فرض بکاپ خودکار ندارد. برای بکاپ‌گیری، از **Snapshotها** استفاده می‌شود که مجموعه‌ای از SSTableها در یک لحظه خاص هستند. این Snapshotها می‌توانند برای بازیابی کامل یا جزئی داده‌ها استفاده شوند.

---

## 📸 ایجاد و بازیابی Snapshot در کاساندرا

### ایجاد Snapshot

برای گرفتن Snapshot از یک کی‌اسپیس، از دستور زیر استفاده کنید:

```bash
nodetool snapshot <keyspace_name> -t <snapshot_name>
```

مثال:

```bash
nodetool snapshot my_keyspace -t snapshot_2025_08_05
```

این دستور یک Snapshot از تمام جداول در کی‌اسپیس `my_keyspace` ایجاد می‌کند و آن را با نام `snapshot_2025_08_05` ذخیره می‌کند.

### بررسی Snapshotها

برای مشاهده لیست Snapshotهای موجود:

```bash
nodetool listsnapshots
```

این دستور اطلاعات Snapshotها، شامل نام، کی‌اسپیس، و اندازه را نمایش می‌دهد.

### بازیابی داده‌ها از Snapshot

برای بازیابی داده‌ها از Snapshot، مراحل زیر را دنبال کنید:

1. **توقف سرویس کاساندرا**:

   ```bash
   sudo service cassandra stop
   ```

2. **حذف داده‌های فعلی**:

   داده‌های موجود در مسیر داده‌ها (مشخص‌شده در `cassandra.yaml` تحت `data_file_directories`) را پاک کنید:

   ```bash
   rm -rf /var/lib/cassandra/data/my_keyspace/*
   ```

3. **کپی Snapshot به مسیر داده‌ها**:

   فایل‌های Snapshot معمولاً در مسیر زیر ذخیره می‌شوند:

   ```
   /var/lib/cassandra/data/<keyspace>/<table>/snapshots/<snapshot_name>/
   ```

   فایل‌ها را به مسیر اصلی داده‌ها کپی کنید:

   ```bash
   cp -r /var/lib/cassandra/data/my_keyspace/mytable/snapshots/snapshot_2025_08_05/* /var/lib/cassandra/data/my_keyspace/mytable/
   ```

4. **راه‌اندازی مجدد سرویس کاساندرا**:

   ```bash
   sudo service cassandra start
   ```

---

## 🐳 دستورات بکاپ و ریکاوری در Docker

وقتی کاساندرا را در Docker اجرا می‌کنید، دستورات باید داخل کانتینر اجرا شوند، و مسیرهای داده به Volume یا Bind Mount متصل هستند.

### گرفتن Snapshot در Docker

برای گرفتن Snapshot داخل کانتینر (فرض کنید نام کانتینر `cassandra-node1` است):

```bash
docker exec -it cassandra-node1 nodetool snapshot my_keyspace -t snapshot_2025_08_05
```

### بررسی Snapshotها در Docker

```bash
docker exec -it cassandra-node1 nodetool listsnapshots
```

### بازیابی از Snapshot در Docker

1. **توقف کانتینر**:

   ```bash
   docker stop cassandra-node1
   ```

2. **حذف داده‌های فعلی**:

   اگر از Volume استفاده می‌کنید، مسیر Volume را بررسی کنید:

   ```bash
   docker volume inspect cassandra_data_volume
   ```

   یا اگر از Bind Mount استفاده می‌کنید (مثلاً `/path/on/host` به `/var/lib/cassandra/data` متصل است)، داده‌ها را پاک کنید:

   ```bash
   rm -rf /path/on/host/my_keyspace/*
   ```

3. **کپی فایل‌های Snapshot**:

   فایل‌های Snapshot را از مسیر بکاپ به مسیر داده‌ها کپی کنید:

   ```bash
   cp -r /backup/cassandra_snapshots/my_keyspace/mytable/snapshot_2025_08_05/* /path/on/host/my_keyspace/mytable/
   ```

4. **راه‌اندازی مجدد کانتینر**:

   ```bash
   docker start cassandra-node1
   ```

### مثال Docker Compose برای کاساندرا

برای ساده‌تر کردن مدیریت، می‌توانید از Docker Compose برای راه‌اندازی یک خوشه کاساندرا استفاده کنید:

```yaml
version: '3.8'
services:
  cassandra:
    image: cassandra:latest
    container_name: cassandra-node1
    volumes:
      - cassandra_data:/var/lib/cassandra
      - ./backups:/backup
    environment:
      - CASSANDRA_CLUSTER_NAME=MyCluster
    ports:
      - "9042:9042"
volumes:
  cassandra_data:
```

این تنظیم یک نود کاساندرا با Volume برای داده‌ها و یک مسیر برای ذخیره بکاپ‌ها ایجاد می‌کند.

---

## 🔧 ابزارها و دستورات مهم برای ریکاوری و نگهداری

| ابزار/دستور | کاربرد |
|--------------|--------|
| `nodetool repair` | همگام‌سازی داده‌ها بین Replicaها برای اطمینان از یکپارچگی |
| `nodetool snapshot` | گرفتن Snapshot برای بکاپ‌گیری |
| `nodetool listsnapshots` | نمایش لیست Snapshotهای موجود |
| `nodetool clearsnapshot` | حذف Snapshotهای قدیمی |
| `nodetool flush` | انتقال داده‌های حافظه (Memtable) به SSTable |
| Commit Log | بازیابی تغییرات ذخیره‌نشده در SSTable |

---

## 🛡️ نکات کلیدی برای استراتژی ریکاوری

- **گرفتن Snapshotهای دوره‌ای**: Snapshotها را به‌صورت منظم بگیرید و در محیطی امن (مثل سرور جداگانه یا Cloud) ذخیره کنید.
- **تنظیم سطح Consistency**: سطح سازگاری (مثل QUORUM یا ONE) را بر اساس نیازهای پروژه تنظیم کنید.
- **اجرای دوره‌ای Repair**: از `nodetool repair` برای همگام‌سازی داده‌ها در خوشه استفاده کنید.
- **مانیتورینگ خوشه**: ابزارهای مانیتورینگ (مثل Prometheus یا DataStax OpsCenter) را برای شناسایی سریع مشکلات فعال کنید.
- **مدیریت Commit Log**: حجم و تعداد Commit Logها را در `cassandra.yaml` تنظیم کنید تا منابع بهینه استفاده شوند.
- **استراتژی بکاپ در محیط تولیدی**: از راهکارهای پیشرفته مانند بکاپ‌گیری افزایشی یا انتقال Snapshot به Cloud (مثل AWS S3) استفاده کنید.

---

## 📚 مثال عملی: بکاپ و ریکاوری

### ۱. گرفتن Snapshot از کی‌اسپیس

```bash
nodetool snapshot my_keyspace -t mybackup_2025_08_05
```

یا در Docker:

```bash
docker exec -it cassandra-node1 nodetool snapshot my_keyspace -t mybackup_2025_08_05
```

### ۲. شناسایی مسیر Snapshot

```bash
ls /var/lib/cassandra/data/my_keyspace/mytable/snapshots/mybackup_2025_08_05/
```

یا در Docker (روی هاست، اگر از Bind Mount استفاده می‌کنید):

```bash
ls /path/on/host/my_keyspace/mytable/snapshots/mybackup_2025_08_05/
```

### ۳. توقف سرویس و پاک کردن داده‌ها

```bash
sudo service cassandra stop
rm -rf /var/lib/cassandra/data/my_keyspace/mytable/*
```

یا در Docker:

```bash
docker stop cassandra-node1
rm -rf /path/on/host/my_keyspace/mytable/*
```

### ۴. کپی فایل‌های Snapshot

```bash
cp -r /var/lib/cassandra/data/my_keyspace/mytable/snapshots/mybackup_2025_08_05/* /var/lib/cassandra/data/my_keyspace/mytable/
```

یا در Docker:

```bash
cp -r /path/on/host/my_keyspace/mytable/snapshots/mybackup_2025_08_05/* /path/on/host/my_keyspace/mytable/
```

### ۵. راه‌اندازی مجدد سرویس

```bash
sudo service cassandra start
```

یا در Docker:

```bash
docker start cassandra-node1
```

---

## 📝 تمرین عملی

برای تسلط بیشتر، این تمرین را انجام دهید:

1. یک کی‌اسپیس نمونه با چند جدول و داده تستی ایجاد کنید.
2. از کی‌اسپیس یک Snapshot بگیرید و آن را در مسیری امن ذخیره کنید.
3. داده‌های فعلی را حذف کنید و با استفاده از Snapshot بازیابی کنید.
4. همین فرآیند را در یک محیط Docker پیاده‌سازی کنید.
5. یک اسکریپت ساده برای خودکارسازی فرآیند بکاپ‌گیری بنویسید (مثلاً با Bash یا Python).

### مثال اسکریپت بکاپ‌گیری خودکار:

```bash
#!/bin/bash
# Script to take a snapshot and move it to a backup directory
KEYSPACE="my_keyspace"
SNAPSHOT_NAME="backup_$(date +%Y_%m_%d)"
BACKUP_DIR="/backup/cassandra_snapshots"

docker exec -it cassandra-node1 nodetool snapshot $KEYSPACE -t $SNAPSHOT_NAME
mkdir -p $BACKUP_DIR/$KEYSPACE
cp -r /path/on/host/$KEYSPACE/*/snapshots/$SNAPSHOT_NAME/* $BACKUP_DIR/$KEYSPACE/
echo "Backup completed for $KEYSPACE: $SNAPSHOT_NAME"
```

---

## 🚀 خلاصه و نکات پایانی

کاساندرا با مکانیزم‌هایی مانند Hinted Handoff، Read Repair، Commit Log، و Snapshot، فرآیندهای بازیابی خودکار و دستی را تسهیل می‌کند. Snapshotها ابزار اصلی برای بکاپ‌گیری دستی هستند، و اجرای دوره‌ای `nodetool repair` برای حفظ یکپارچگی داده‌ها ضروری است. در محیط‌های Docker، دستورات مشابه هستند اما باید داخل کانتینر اجرا شوند. با تنظیم سطح Consistency مناسب و استفاده از ابزارهای مانیتورینگ، می‌توانید یک استراتژی ریکاوری مطمئن ایجاد کنید.

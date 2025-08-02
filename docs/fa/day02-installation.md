
# روز 02 - نصب Apache Cassandra روی لینوکس و ویندوز
![Cassandra Logo](https://img.shields.io/badge/Apache%20Cassandra-1287B1?style=flat&logo=apache-cassandra&logoColor=white) 
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/mrkeshi/cassandra-journey-10days?style=social)](https://github.com/mrkeshi/cassandra-journey-10days)

این راهنما آموزش کامل و گام‌به‌گام نصب Apache Cassandra را روی سیستم‌عامل‌های **لینوکس (Debian/Ubuntu و CentOS/RHEL)** و **ویندوز** با استفاده از WSL یا داکر ارائه می‌دهد.

---

## نصب Apache Cassandra روی لینوکس

### پیش‌نیازها

- یک سیستم لینوکسی (Debian/Ubuntu یا CentOS/RHEL)
- نصب Java نسخه ۸ یا ۱۱
- نصب curl یا wget
- دسترسی sudo
- اتصال اینترنت

### نصب روی Ubuntu/Debian

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version

curl https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor -o /usr/share/keyrings/cassandra.gpg

echo "deb [signed-by=/usr/share/keyrings/cassandra.gpg] https://downloads.apache.org/cassandra/debian 40x main" | sudo tee /etc/apt/sources.list.d/cassandra.list

sudo apt update
sudo apt install cassandra -y

sudo systemctl enable cassandra
sudo systemctl start cassandra
sudo systemctl status cassandra

cqlsh
```

### نصب روی CentOS/RHEL

```bash
sudo yum install java-11-openjdk -y
java -version

sudo tee /etc/yum.repos.d/cassandra.repo <<EOF
[cassandra]
name=Apache Cassandra
baseurl=https://downloads.apache.org/cassandra/redhat/40x/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://downloads.apache.org/cassandra/KEYS
EOF

sudo yum install cassandra -y

sudo systemctl enable cassandra
sudo systemctl start cassandra
sudo systemctl status cassandra

cqlsh
```

---

## نصب Apache Cassandra روی ویندوز (با WSL و Docker)

این راهنما نصب Apache Cassandra را روی ویندوز با دو روش زیر آموزش می‌دهد:

- [۱] استفاده از **WSL (Windows Subsystem for Linux)**
- [۲] استفاده از **Docker**

---

### پیش‌نیازها

- ویندوز ۱۰ یا ۱۱
- فعال بودن WSL (برای روش اول)
- نصب Docker Desktop (برای روش دوم)
- اتصال اینترنت پایدار

---

### روش اول: نصب Cassandra با WSL

این روش اجرای Cassandra مبتنی بر لینوکس را در محیط WSL ویندوز فراهم می‌کند.

#### مرحله ۱: نصب WSL و Ubuntu

PowerShell را با دسترسی Administrator باز کرده و دستور زیر را اجرا کنید:

```powershell
wsl --install
```

این دستور WSL و توزیع پیش‌فرض اوبونتو را نصب می‌کند. اگر قبلاً نصب شده، با دستور زیر به‌روزرسانی کنید:

```powershell
wsl --update
```

در صورت درخواست سیستم را ریستارت کنید.

---

#### مرحله ۲: اجرای Ubuntu و نصب Cassandra

ترمینال Ubuntu را از منوی استارت باز کرده و دستورات زیر را اجرا کنید:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

نسخه جاوا را بررسی کنید:

```bash
java -version
```

مخزن Cassandra را اضافه کرده و نصب کنید:

```bash
curl https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor -o /usr/share/keyrings/cassandra.gpg

echo "deb [signed-by=/usr/share/keyrings/cassandra.gpg] https://downloads.apache.org/cassandra/debian 40x main" | sudo tee /etc/apt/sources.list.d/cassandra.list

sudo apt update
sudo apt install cassandra -y
```

---

#### مرحله ۳: شروع Cassandra و تست

```bash
sudo systemctl start cassandra
cqlsh
```

اگر پرامپت `cqlsh` ظاهر شد، نصب Cassandra داخل WSL موفقیت‌آمیز بوده است.

---

### روش دوم: نصب Cassandra با Docker

این روش ساده‌ترین و قابل حمل‌ترین راه اجرای Cassandra روی ویندوز است.

#### مرحله ۱: نصب Docker Desktop

Docker Desktop برای ویندوز را از لینک زیر دانلود و نصب کنید:  
👉 https://www.docker.com/products/docker-desktop/

اطمینان حاصل کنید که Docker در حال اجرا است.

---

#### مرحله ۲: اجرای کانتینر Cassandra

در یک پنجره PowerShell یا CMD، دستورات زیر را اجرا کنید:

```bash
docker pull cassandra
docker run --name cassandra -p 9042:9042 -d cassandra
```

این کانتینر Cassandra را اجرا می‌کند و پورت پیش‌فرض CQL (9042) را به میزبان متصل می‌کند.

---

#### مرحله ۳: اتصال به Cassandra

از CQL shell داخل کانتینر استفاده کنید:

```bash
docker exec -it cassandra cqlsh
```

باید پرامپت `cqlsh>` را ببینید. شما اکنون به نمونه Cassandra خود در Docker متصل شده‌اید.

---

### نکات و توصیه‌ها

- داده‌های Cassandra در Docker **موقتی** هستند مگر اینکه حجم‌ها (volumes) پیکربندی شوند.
- در WSL، می‌توانید Cassandra را همانند یک سیستم اوبونتو مدیریت کنید.
- می‌توانید ابزارهای گرافیکی مانند [DataStax Studio](https://www.datastax.com/studio) را برای مشاهده داده‌ها نصب کنید.

---

### منابع

- [مستندات رسمی Apache Cassandra](https://cassandra.apache.org/doc/latest/)
- [Docker Hub: تصویر cassandra](https://hub.docker.com/_/cassandra)
- [راهنمای Microsoft WSL](https://learn.microsoft.com/en-us/windows/wsl/)

# Docker Compose Services

รวม Services ทั้งหมดที่ใช้งานใน `docker-compose.yml`

## สารบัญ

- [การเริ่มต้นใช้งาน](#การเริ่มต้นใช้งาน)
- [PostgreSQL](#postgresql)
- [pgAdmin](#pgadmin)
- [MinIO](#minio)
- [RabbitMQ](#rabbitmq)
- [Redis](#redis)
- [คำสั่งที่ใช้บ่อย](#คำสั่งที่ใช้บ่อย)

---

## การเริ่มต้นใช้งาน

```bash
# เริ่มต้น services ทั้งหมด
docker compose up -d

# หยุด services ทั้งหมด
docker compose down

# ดู logs ทั้งหมด
docker compose logs -f

# ดู logs เฉพาะ service
docker compose logs -f <service_name>
```

---

## PostgreSQL

Database หลักสำหรับเก็บข้อมูล

| รายการ | ค่า |
|---|---|
| Image | `postgres:17` |
| Port | `5432` |
| Username | `admin` |
| Password | `password` |
| Database | `mydatabase` |

### การเชื่อมต่อ

```
# Connection String
postgresql://admin:password@localhost:5432/mydatabase

# psql
psql -h localhost -p 5432 -U admin -d mydatabase
```

### เข้า shell ของ container

```bash
docker exec -it postgres psql -U admin -d mydatabase
```

### Backup & Restore

```bash
# Backup
docker exec -it postgres pg_dump -U admin mydatabase > ./backup/backup.sql

# Restore
docker exec -i postgres psql -U admin -d mydatabase < ./backup/backup.sql
```

### Log files

Log ถูกเก็บไว้ที่ `./logs/` ในรูปแบบ `postgresql-YYYY-MM-DD_HHMMSS.log`

---

## pgAdmin

Web UI สำหรับจัดการ PostgreSQL

| รายการ | ค่า |
|---|---|
| Image | `dpage/pgadmin4` |
| URL | http://localhost:5050 |
| Email | `admin@admin.com` |
| Password | `password` |

### วิธีเชื่อมต่อกับ PostgreSQL จาก pgAdmin

1. เปิด http://localhost:5050
2. Login ด้วย email/password ด้านบน
3. Add New Server:
   - **Name:** ตั้งชื่อตามต้องการ
   - **Host:** `postgres` (ใช้ชื่อ container)
   - **Port:** `5432`
   - **Username:** `admin`
   - **Password:** `password`

---

## MinIO

Object Storage ที่เข้ากันได้กับ Amazon S3

| รายการ | ค่า |
|---|---|
| Image | `minio/minio` |
| API Port | `9000` |
| Console UI | http://localhost:9001 |
| Username | `admin` |
| Password | `password` |
| Memory Limit | 512M |

### การใช้งาน Console

1. เปิด http://localhost:9001
2. Login ด้วย username/password ด้านบน
3. สร้าง Bucket และ upload ไฟล์ได้จากหน้า Console

### การเชื่อมต่อจาก Application

```
Endpoint:  http://localhost:9000
Access Key: admin
Secret Key: password
```

### ใช้งานผ่าน mc (MinIO Client)

```bash
# ตั้งค่า alias
mc alias set local http://localhost:9000 admin password

# สร้าง bucket
mc mb local/my-bucket

# upload ไฟล์
mc cp myfile.txt local/my-bucket/

# list ไฟล์
mc ls local/my-bucket/
```

---

## RabbitMQ

Message Broker สำหรับ Queue/Pub-Sub

| รายการ | ค่า |
|---|---|
| Image | `rabbitmq:4-management` |
| AMQP Port | `5672` |
| Management UI | http://localhost:15672 |
| Username | `admin` |
| Password | `password` |
| Memory Limit | 512M |

### การใช้งาน Management UI

1. เปิด http://localhost:15672
2. Login ด้วย username/password ด้านบน
3. จัดการ Queues, Exchanges, Bindings ได้จากหน้า UI

### การเชื่อมต่อจาก Application

```
# AMQP URL
amqp://admin:password@localhost:5672
```

### ตัวอย่างการใช้งาน (Node.js - amqplib)

```javascript
const amqp = require('amqplib');

const conn = await amqp.connect('amqp://admin:password@localhost:5672');
const channel = await conn.createChannel();

// สร้าง queue
await channel.assertQueue('my-queue');

// ส่ง message
channel.sendToQueue('my-queue', Buffer.from('Hello'));

// รับ message
channel.consume('my-queue', (msg) => {
  console.log(msg.content.toString());
  channel.ack(msg);
});
```

---

## Redis

In-Memory Data Store สำหรับ Cache / Session / Pub-Sub

| รายการ | ค่า |
|---|---|
| Image | `redis:7-alpine` |
| Port | `6379` |
| Password | `password` |
| Persistence | AOF (Append Only File) |
| Memory Limit | 256M |

### การเชื่อมต่อ

```
# Connection String
redis://:password@localhost:6379
```

### เข้า redis-cli ผ่าน container

```bash
docker exec -it redis redis-cli -a password
```

### คำสั่ง Redis พื้นฐาน

```bash
# ตั้งค่า key
SET mykey "Hello"

# อ่านค่า key
GET mykey

# ตั้งค่าพร้อม expiry (วินาที)
SET session:abc "data" EX 3600

# ลบ key
DEL mykey

# ดู keys ทั้งหมด
KEYS *

# ดูข้อมูล server
INFO
```

### ตัวอย่างการใช้งาน (Node.js - ioredis)

```javascript
const Redis = require('ioredis');

const redis = new Redis({
  host: 'localhost',
  port: 6379,
  password: 'password',
});

await redis.set('key', 'value');
const val = await redis.get('key');
console.log(val); // "value"
```

---

## คำสั่งที่ใช้บ่อย

```bash
# เริ่ม services ทั้งหมด
docker compose up -d

# หยุด services ทั้งหมด
docker compose down

# restart service เดียว
docker compose restart <service_name>

# ดู status ของทุก services
docker compose ps

# ดู resource usage
docker stats

# เข้า shell ของ container
docker exec -it <container_name> bash
```

---

## Ports Summary

| Service | Port | URL |
|---|---|---|
| PostgreSQL | 5432 | - |
| pgAdmin | 5050 | http://localhost:5050 |
| MinIO API | 9000 | - |
| MinIO Console | 9001 | http://localhost:9001 |
| RabbitMQ AMQP | 5672 | - |
| RabbitMQ UI | 15672 | http://localhost:15672 |
| Redis | 6379 | - |

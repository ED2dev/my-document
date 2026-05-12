# 🐘 PostgreSQL Cheatsheet

> Tổng hợp các câu lệnh PostgreSQL thường dùng trên Terminal và DBeaver.

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![DBeaver](https://img.shields.io/badge/DBeaver-382923?style=for-the-badge&logo=dbeaver&logoColor=white)
![Terminal](https://img.shields.io/badge/Terminal-000000?style=for-the-badge&logo=gnometerminal&logoColor=white)

---

## 📂 Mục lục

- [Terminal](#-terminal)
  - [Kết nối](#kết-nối)
  - [Database](#database)
  - [Schema](#schema)
  - [Table](#table)
- [DBeaver](#-dbeaver)
- [Bảng tổng hợp lệnh psql](#-bảng-tổng-hợp-lệnh-psql)

---

## 💻 Terminal

### Kết nối

```bash
# Đăng nhập vào PostgreSQL
psql -U postgres

# Kiểm tra version
SELECT version();

# Thoát khỏi psql
\q
```

### Database

```bash
# Tạo database
CREATE DATABASE db_name;

# Xem danh sách database
\l

# Kết nối tới database
\c db_name
```

### Schema

```sql
-- Tạo schema
CREATE SCHEMA inventory;

-- Xem danh sách schema
\dn

-- Xóa schema (đảm bảo đã xóa hết table bên trong)
DROP SCHEMA schema_name;

-- Xóa schema và toàn bộ object bên trong
DROP SCHEMA schema_name CASCADE;

-- Xóa schema an toàn (không báo lỗi nếu không tồn tại)
DROP SCHEMA IF EXISTS schema_name CASCADE;
```

### Table

#### Xem danh sách table

```bash
# Xem toàn bộ table
\dt

# Xem table của schema cụ thể
\dt schema_name.*

# Xem cấu trúc của 1 table
\d table_name
```

#### Tạo table

```sql
-- Tạo table đơn giản
CREATE TABLE test_table (
    id   SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Tạo table trong schema cụ thể
CREATE TABLE inventory.products (
    id    SERIAL PRIMARY KEY,
    name  VARCHAR(100),
    price NUMERIC(10, 2)
);
```

#### Xóa table

```sql
-- Xóa table
DROP TABLE table_name;

-- Xóa table trong schema cụ thể
DROP TABLE schema_name.table_name;

-- Xóa table an toàn (không báo lỗi nếu không tồn tại)
DROP TABLE IF EXISTS schema_name.table_name;
```

---

## 🖥️ DBeaver

### Kết nối PostgreSQL

```
Connections → Create → Connection → PostgreSQL
```

| Trường | Giá trị |
|--------|---------|
| Host | `localhost` |
| Port | `5432` |
| Database | `db_name` |
| Username | `postgres` |
| Password | _(mật khẩu bạn đặt)_ |

> 💡 Nhấn **"Test Connection"** trước khi lưu để kiểm tra kết nối thành công.

---

## 📌 Bảng tổng hợp lệnh psql

| Lệnh | Mô tả |
|------|-------|
| `\l` | Xem danh sách database |
| `\c db_name` | Kết nối tới database |
| `\dn` | Xem danh sách schema |
| `\dt` | Xem danh sách table |
| `\dt schema.*` | Xem table theo schema |
| `\d table_name` | Xem cấu trúc table |
| `\q` | Thoát psql |

---

> 📌 Tài liệu được cập nhật liên tục theo quá trình học.

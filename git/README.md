# 🐘 PostgreSQL Statement

> Tài liệu cá nhân về các câu lệnh PostgreSQL thường dùng — từ cơ bản đến nâng cao.

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![Status](https://img.shields.io/badge/status-updating-yellow?style=for-the-badge)

---

## 📂 Mục lục

- [Kết nối](#-kết-nối)
- [DDL — Định nghĩa dữ liệu](#-ddl--định-nghĩa-dữ-liệu)
- [DML — Thao tác dữ liệu](#-dml--thao-tác-dữ-liệu)
- [Query nâng cao](#-query-nâng-cao)
- [Index](#-index)
- [Transaction](#-transaction)

---

## 🔌 Kết nối

```bash
# Kết nối vào PostgreSQL
psql -U username -d database_name

# Kết nối với host
psql -h localhost -p 5432 -U username -d database_name
```

---

## 🏗️ DDL — Định nghĩa dữ liệu

### Tạo bảng

```sql
CREATE TABLE users (
    id        SERIAL PRIMARY KEY,
    name      VARCHAR(100) NOT NULL,
    email     VARCHAR(150) UNIQUE NOT NULL,
    age       INT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Sửa bảng

```sql
-- Thêm cột
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Đổi tên cột
ALTER TABLE users RENAME COLUMN phone TO phone_number;

-- Xóa cột
ALTER TABLE users DROP COLUMN phone_number;
```

### Xóa bảng

```sql
DROP TABLE users;
DROP TABLE IF EXISTS users; -- Không báo lỗi nếu không tồn tại
```

---

## ✏️ DML — Thao tác dữ liệu

### INSERT

```sql
-- Thêm 1 dòng
INSERT INTO users (name, email, age)
VALUES ('Nguyen Van A', 'a@gmail.com', 25);

-- Thêm nhiều dòng
INSERT INTO users (name, email, age)
VALUES
    ('Tran Thi B', 'b@gmail.com', 30),
    ('Le Van C',   'c@gmail.com', 22);
```

### SELECT

```sql
-- Lấy tất cả
SELECT * FROM users;

-- Lấy có điều kiện
SELECT name, email FROM users
WHERE age > 18
ORDER BY name ASC
LIMIT 10;
```

### UPDATE

```sql
UPDATE users
SET age = 26
WHERE id = 1;
```

### DELETE

```sql
DELETE FROM users WHERE id = 1;

-- Xóa toàn bộ dữ liệu (giữ bảng)
TRUNCATE TABLE users;
```

---

## 🔍 Query nâng cao

### JOIN

```sql
-- INNER JOIN
SELECT u.name, o.product
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN
SELECT u.name, o.product
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### GROUP BY & HAVING

```sql
SELECT age, COUNT(*) AS total
FROM users
GROUP BY age
HAVING COUNT(*) > 1
ORDER BY age;
```

### Subquery

```sql
SELECT name FROM users
WHERE id IN (
    SELECT user_id FROM orders WHERE total > 100
);
```

### CTE (Common Table Expression)

```sql
WITH active_users AS (
    SELECT * FROM users WHERE status = 'active'
)
SELECT * FROM active_users WHERE age > 18;
```

---

## ⚡ Index

```sql
-- Tạo index
CREATE INDEX idx_users_email ON users(email);

-- Tạo unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Xem danh sách index
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'users';

-- Xóa index
DROP INDEX idx_users_email;
```

---

## 🔒 Transaction

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;   -- Lưu thay đổi
-- hoặc
ROLLBACK; -- Hủy thay đổi nếu có lỗi
```

---

## 🚀 Push code lên GitHub

```bash
git add .
git commit -m "your message"
git push origin main
```

---

> 📌 Tài liệu được cập nhật liên tục theo quá trình học.

# Deploy Odoo 17 bằng Docker (Docker Compose) – Step by Step

Tài liệu này hướng dẫn **triển khai Odoo 17 + PostgreSQL bằng Docker Compose** theo thứ tự chuẩn, làm theo là chạy ngay. Phù hợp **local dev / test**.

---

## 0. Chuẩn bị

- Cài đặt:
    - Docker
    - Docker Compose (plugin `docker compose`)
- Kiểm tra:
```bash
docker --version
docker compose version
text
# Hướng dẫn cài đặt Odoo 17 với Docker (Development/Test)

## 1. Tạo cấu trúc thư mục
mkdir -p odoo17/{config,addons,logs}
cd odoo17

text

**Cấu trúc thư mục:**
odoo17/
├── docker-compose.yml
├── config/
│ └── odoo.conf
├── addons/
└── logs/

text

## 2. Tạo file `docker-compose.yml`
services:
db:
image: postgres:15
container_name: odoo17-db
environment:
POSTGRES_USER: odoo
POSTGRES_PASSWORD: odoo
POSTGRES_DB: postgres
volumes:
- odoo17-db-data:/var/lib/postgresql/data
restart: unless-stopped

odoo:
image: odoo:17.0
container_name: odoo17
depends_on:
- db
ports:
- "8069:8069" # Web UI
- "8072:8072" # Longpolling / Live chat
environment:
HOST: db
USER: odoo
PASSWORD: odoo
volumes:
- odoo17-web-data:/var/lib/odoo
- ./config/odoo.conf:/etc/odoo/odoo.conf:ro
- ./addons:/mnt/extra-addons
- ./logs:/var/log/odoo
restart: unless-stopped

volumes:
odoo17-db-data:
odoo17-web-data:

text

## 3. Tạo file cấu hình `config/odoo.conf`
[options]
admin_passwd = admin

db_host = db
db_port = 5432
db_user = odoo
db_password = odoo

addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons

proxy_mode = True
logfile = /var/log/odoo/odoo.log

text

## 4. Chạy Odoo
docker compose up -d
docker compose ps

text

## 5. Tạo database lần đầu
Mở trình duyệt: [**http://localhost:8069**](http://localhost:8069)

Tại màn hình **Create Database**:
- **Master Password**: `admin` (phải trùng `admin_passwd`)
- **Database Name**: ví dụ `odoo17`
- **Email / Password**: tài khoản admin Odoo
- **(Dev)** Tick **Demo data** nếu muốn
- Bấm **Create database**
- ⏳ **Đợi 1–2 phút** để khởi tạo.

## 6. Kiểm tra log khi có lỗi
docker logs -f odoo17
docker logs -f odoo17-db

text

## 7. Dừng / khởi động lại
docker compose stop
docker compose start
docker compose restart

text

## 8. Xóa toàn bộ để làm lại từ đầu **(CẨN THẬN – mất dữ liệu)**
docker compose down -v

text

## 📌 Ghi chú quan trọng
- **Master Password (admin_passwd) ≠ mật khẩu đăng nhập Odoo**
- **Thư mục `addons/`**: nơi đặt custom modules
- **Cấu hình này chỉ phù hợp dev/test**, chưa an toàn cho production

## 🚀 Gợi ý tiếp theo (Production)
- Thêm **Nginx reverse proxy**
- Chặn `/web/database/*`
- Bật **SSL (Let's Encrypt)**
- Thiết lập **backup/restore PostgreSQL*

## thong nhat mo hinh lam viec gitlab

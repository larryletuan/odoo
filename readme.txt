0) Chuẩn bị

Cài Docker + Docker Compose plugin.

1) Tạo thư mục dự án
mkdir -p odoo17/{config,addons,logs}
cd odoo17

2) Tạo docker-compose.yml

Tạo file docker-compose.yml với nội dung:

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
      - "8069:8069"
      - "8072:8072"
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

3) Tạo config/odoo.conf (fix Master Password cho khỏi cảnh báo)

Tạo file config/odoo.conf:

[options]
admin_passwd = admin

db_host = db
db_port = 5432
db_user = odoo
db_password = odoo

addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons

proxy_mode = True
logfile = /var/log/odoo/odoo.log

4) Chạy container
docker compose up -d
docker compose ps

5) Mở Odoo và tạo database

Mở trình duyệt:

http://localhost:8069

Tại màn hình Create Database:

Master Password: admin (đúng như admin_passwd)

Database Name: ví dụ odoo17

Email/Password: tài khoản admin Odoo bạn muốn tạo

(Dev) tick Demo data nếu muốn

Bấm Create database

6) Nếu bị lỗi (cách kiểm tra nhanh)

Xem log:

docker logs -f odoo17
docker logs -f odoo17-db

7) Dừng / chạy lại
docker compose stop
docker compose start
# hoặc restart
docker compose restart

8) Xoá sạch để làm lại từ đầu (cẩn thận, mất dữ liệu)
docker compose down -v
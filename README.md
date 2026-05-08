# baitap2-opensource-django
# Nguyễn Hoàng Việt K22548016074
# 1. CSDL cho tiệm cầm đồ:

<img width="296" height="591" alt="image" src="https://github.com/user-attachments/assets/1d351c49-ded2-412b-a9d5-d9b09c824274" />

# 2. Cấu trúc thư mục 

<img width="474" height="402" alt="image" src="https://github.com/user-attachments/assets/d60e825c-6017-4123-9a8c-0a7e3458438b" />

# 3. Tạo các file cấu hình

File này hướng dẫn Docker cách build image cho Django:

```dockerfile
FROM python:3.11-slim

Cài thư viện hệ thống để build mysqlclient

RUN apt-get update && apt-get install -y \
    default-libmysqlclient-dev pkg-config gcc \
    && rm -rf /var/lib/apt/lists/*

Thư mục làm việc trong container
WORKDIR /app

Copy requirements trước để tận dụng Docker cache
COPY requirements.txt .

Cài thư viện Python
RUN pip install --no-cache-dir -r requirements.txt

Mở cổng 8000
EXPOSE 8000

Khởi động Django (0.0.0.0 để nhận từ bên ngoài container)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

# requirements.txt

<img width="660" height="75" alt="image" src="https://github.com/user-attachments/assets/c500271c-34e8-4117-a6fc-fbe03fa08d44" />

# Docker-compose.yml định nghĩa 3 service 

version: '3.8'
services:
  db:
    image: mariadb:10.11
    container_name: tcd_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: tiem_cam_do
      MYSQL_USER: django_user
      MYSQL_PASSWORD: django123
    volumes:
      - db_data:/var/lib/mysql   # Lưu data bền vững
    networks: [tcd_net]

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: tcd_phpmyadmin
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: root123
    ports: ['8080:80']
    depends_on: [db]
    networks: [tcd_net]

  web:
    build: ./django_app         # Build từ Dockerfile
    container_name: tcd_django
    volumes:
      - ./django_app:/app        # Mount để edit trực tiếp
    ports: ['8000:8000']
    depends_on: [db]
    networks: [tcd_net]
    environment:
      DB_NAME: tiem_cam_do
      DB_USER: django_user
      DB_PASSWORD: django123
      DB_HOST: db
      DB_PORT: '3306'

volumes:
  db_data:
networks:
  tcd_net:
  
# 4. Cấu hình Django 
# Khởi tạo 
cd ~/tiem_cam_do

Build image
docker compose build

Tạo Django project
docker compose run --rm web django-admin startproject config .

Tạo app core
docker compose run --rm web python manage.py startapp core

Cấp quyền chỉnh sửa file (do Docker tạo với quyền root)
sudo chown -R viet123:viet123 ~/tiem_cam_do/django_app/

# Sửa file settings.py 

<img width="1133" height="982" alt="image" src="https://github.com/user-attachments/assets/fb4c42dc-bf45-41f2-a646-02501165f239" />

# Thiết kế csdl (model.py)

<img width="953" height="987" alt="image" src="https://github.com/user-attachments/assets/11baa5eb-38b9-40f7-87b1-2941a4893ae4" />

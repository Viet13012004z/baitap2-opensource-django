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
```
# 4. Cấu hình Django 
# Khởi tạo 
```
cd ~/tiem_cam_do

# Build image
docker compose build

# Tạo Django project
docker compose run --rm web django-admin startproject config .

# Tạo app core
docker compose run --rm web python manage.py startapp core

# Cấp quyền chỉnh sửa file (do Docker tạo với quyền root)
sudo chown -R viet123:viet123 ~/tiem_cam_do/django_app/
```
# Sửa file settings.py 

<img width="1133" height="982" alt="image" src="https://github.com/user-attachments/assets/fb4c42dc-bf45-41f2-a646-02501165f239" />

# Thiết kế csdl (model.py)

<img width="953" height="987" alt="image" src="https://github.com/user-attachments/assets/11baa5eb-38b9-40f7-87b1-2941a4893ae4" />

# Trang admin và trang con nợ đến hạn 
# Cấu hình admin.py 
```
from django.contrib import admin
from .models import KhachHang, HopDongCam, ...  # import tất cả

@admin.register(HopDongCam)
class HopDongCamAdmin(admin.ModelAdmin):
    # Các cột hiển thị trong danh sách
    list_display = ['id', 'khach_hang', 'san_pham', 'ngay_dao_han', 'trang_thai']
    list_filter = ['trang_thai']      # Lọc nhanh theo trạng thái
    search_fields = ['khach_hang__ho_ten']  # Tìm kiếm theo tên KH
    date_hierarchy = 'ngay_dao_han'   # Điều hướng theo ngày
```
# Cấu hình views.py + home.html 
```
def home_page(request):
    hom_nay = timezone.localdate()
    con_no = HopDongCam.objects.filter(
        ngay_dao_han__lte=hom_nay,   # Ngày đáo hạn <= hôm nay
        trang_thai='dang_cam'         # Chưa chuộc
    ).select_related('khach_hang', 'san_pham')
     .order_by('ngay_dao_han')        # Sắp xếp quá hạn lâu nhất lên đầu
    return render(request, 'core/home.html', {'con_no': con_no, 'hom_nay': hom_nay})
```
# Template home.html dùng cú pháp Jinja2 (Django template):
```
{% for hd in con_no %}
<tr>
  <td>{{ hd.khach_hang.ho_ten }}</td>
  <td>{{ hd.tien_cho_vay|floatformat:0 }}đ</td>
  <td>{{ hd.ngay_dao_han|timesince }} trước</td>  {# Tính số ngày quá hạn #}
</tr>
{% endfor %}
```
# Chạy và khởi tạo hệ thống 
Dưới đây là các bước thiết lập hệ thống của bạn được chuyển thành định dạng bảng Markdown:

### Hướng dẫn thiết lập hệ thống Docker & Django

| Bước | Hành động | Lệnh thực thi / Đường dẫn |
| --- | --- | --- |
| **1** | Khởi động 3 container | `docker compose up -d` |
| **2** | Tạo migration từ models | `docker compose exec web python manage.py makemigrations core` |
| **3** | Áp dụng migration vào DB | `docker compose exec web python manage.py migrate` |
| **4** | Tạo tài khoản admin | `docker compose exec web python manage.py createsuperuser` |
| **5** | Truy cập trang Admin | [http://192.168.11.101:8000/admin](http://192.168.11.101:8000/admin) |
| **6** | Xem CSDL (phpMyAdmin) | [http://192.168.11.101:8080](http://192.168.11.101:8080) |

# Clouldfare tunnel
Vào https://one.cloudflare.com → Networks → Tunnels → Create a tunnel

<img width="1918" height="751" alt="image" src="https://github.com/user-attachments/assets/f470e1cd-022c-45fc-b8da-c3280bb2f399" />

Đặt tên tunnel: tiem-cam-do → Save tunnel

Chọn OS: Debian (không chọn Windows!) → copy lệnh 'Install as service'

<img width="1148" height="773" alt="image" src="https://github.com/user-attachments/assets/994949a6-ebc7-42d1-967c-5329bd14b688" />

Paste lệnh vào SSH Ubuntu chạy → Dashboard hiện 'Tunnel connected successfully'
Bấm Continue → Add public hostname: subdomain=camdo, domain=..., URL=http://localhost:8000
Bấm Save tunnel

# Kết quả hiển thị: 
Giao diện trang admin và các tính năng 

<img width="944" height="2046" alt="image" src="https://github.com/user-attachments/assets/fd0e6055-fd49-47d0-bb00-86633252139d" />

Giao diện trang thông báo những con nợ trễ hạn 

<img width="944" height="2046" alt="image" src="https://github.com/user-attachments/assets/7741a31d-1e13-49a6-b3ef-e55ccbbff554" />

# Kết quả đạt được 
| STT | Yêu cầu | Kết quả |
|:---:|:---|:---:|
| 1 | 3 service Docker (MariaDB, phpMyAdmin, Django) | ✅ Hoàn thành |
| 2 | Django kết nối MariaDB, đọc từ settings.py | ✅ Hoàn thành |
| 3 | Trang Admin đăng nhập, thêm/sửa/xoá dữ liệu | ✅ Hoàn thành |
| 4 | FK hiển thị tên, lưu ID vào DB (kiểm tra qua phpMyAdmin) | ✅ Hoàn thành |
| 5 | Template Jinja2 - trang con nợ đến hạn | ✅ Hoàn thành |
| 6 | Public qua Cloudflare Tunnel với subdomain thật | ✅ Hoàn thành |

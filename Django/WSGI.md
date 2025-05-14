### **1. WSGI là gì?**

- **WSGI (Web Server Gateway Interface)** là một tiêu chuẩn giúp Python web frameworks (như Django) giao tiếp với web servers (như Gunicorn, uWSGI, Apache mod_wsgi).
- Django triển khai WSGI thông qua một **callable object** tên là `application`.
### **2. `wsgi.py` và `application` callable**

- Khi tạo một Django project bằng:
```bash
django-admin startproject myproject
```
* Django sẽ tự động tạo file `wsgi.py`:
```sh
myproject/
├── wsgi.py
├── settings.py
├── urls.py
├── asgi.py
├── ...
```
- Trong `wsgi.py`, Django định nghĩa `application`:
```python
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = get_wsgi_application()
```
* `get_wsgi_application()`: Trả về một **WSGI application callable** để server sử dụng.
- **`os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')`**: Xác định file settings mặc định.
### **3. WSGI hoạt động thế nào trong Django?**

#### **3.1. Chạy development server**

- Khi chạy:
```bash
python manage.py runserver
```
* Lệnh `runserver` sẽ đọc **biến cấu hình `WSGI_APPLICATION`** trong `settings.py`:
```python
WSGI_APPLICATION = 'myproject.wsgi.application
```
* Điều này giúp Django biết `application` nằm trong `wsgi.py`.
#### **3.2. Chạy production server**
- Trong môi trường production, một WSGI server như **Gunicorn** hoặc **uWSGI** sẽ chạy ứng dụng:
```bash
gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
```
* `myproject.wsgi:application`: Trỏ đến `application` trong `wsgi.py`.
### **4. Cấu hình `DJANGO_SETTINGS_MODULE`**

- Khi Django khởi động, nó cần biết **file settings nào cần dùng**.
- `DJANGO_SETTINGS_MODULE` là biến môi trường chứa đường dẫn đến settings module, ví dụ:
```python
export DJANGO_SETTINGS_MODULE=myproject.settings
```
- Nếu không đặt biến này, `wsgi.py` sẽ tự động thiết lập:
```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')
```
- Trong production, có thể sử dụng các settings khác nhau:
```python
export DJANGO_SETTINGS_MODULE=myproject.settings.production
gunicorn myproject.wsgi:application
```
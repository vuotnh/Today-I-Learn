* `settings.py` là file cấu hình bao gồm toàn bộ các cấu hình của django
* `settings.py` là một <span style="color:rgb(255, 0, 0)">python module</span>, đơn giản là một file `.py` chứ code python, các cấu hình trong file này là các ***biến ở cấp module***
* Ví dụ, vì là module nên ta có thể import file này giống bất kỳ module nào khác
```python
import mysite.settings
print(mysite.settins.DEBUG)
```

Ví dụ về cấu hình trong setting
```python
ALLOWED_HOSTS = ["www.example.com"]
DEBUG = False
DEFAULT_FROM_EMAIL = "webmaster@example.com"
MY_SETTING = [str(i) for i in range(30)]
```

*Khi sử dụng Django, ta cần chỉ định đường dẫn của file settings.py mà ta muốn sử dụng trong biến môi trường `DJANGO_SETTING_MODULE`*

## Default settings
Django không yêu cầu bắt buộc phải định nghĩa tất cả các settings trong file `settings.py`, các giá trị default setting của django được định nghĩa trong `django/conf/global_settings.py`.

***Cách django load settings***
1. Django sẽ load các default setting từ file `django/conf/global_settings.py`
2. Load các setting từ file `settings.py` của project. Nếu có config nào được khai báo trong `settings.py`, nó sẽ ghi đè lên các giá trị mặc định được load từ `global_settings.py`

Có thể chạy lệnh sau để xem sự khác nhau giữa cấu hình hiện tại và cấu hình default
```bash
python manage.py diffsettings
```

## Sử dụng các setting trong code
```python
from django.conf import settings
from django.conf.settings import DEBUG
if settings.DEBUG:
	pass
```

***Ghi đè các setting trong runtime*** (không khuyến khích sử dụng)
```python
from django.conf import settings

settings.DEBUG = True
```

## Custom setting
Django cho phép cấu hình ***settings một cách thủ công*** bằng cách gọi hàm `settings.configure()`
```python
from django.conf import settings

# Cấu hình settings thủ công mà không cần file settings.py
settings.configure(DEBUG=True, COMPANY_NAME="omzcloud")
'''
Đoạn code trên bật chế độ debug, thiết lập config COMPANY_NAME là omzcloud
'''
```

### Custom default setting
Mặc định, django sử dụng các config trong `django.conf.global_settings` làm `default_settings`. Tuy nhiên có thể thay đổi các default settings này bằng cách truyền một module hoặc class vào `default_setting` khi dùng `settings.configure()`
```python
# mysite/my_default_settings.py

# Các giá trị mặc định của ứng dụng
DATABASE_ENGINE = "sqlite3"
ALLOWED_HOSTS = ["localhost"]
LOG_LEVEL = "WARNING
```

```python
from django.conf import settings
from mysite import my_default_settings  # Import module chứa default settings

# Cấu hình Django với my_default_settings và ghi đè DEBUG
settings.configure(default_settings=my_default_settings, DEBUG=True)
```

> <span style="color: red">Lưu ý</span>: Django chỉ cho phép dùng 1 trong 2 cách để thiệt lập settings. Một là sử dụng biến môi trường `DJANGO_SETTING_MODULES` hoặc là gọi `settings.configure` một lần trước khi truy cập settings

```python
'''
Lỗi ImportError: Requested setting DEBUG, but settings are not configured do Django không biết lấy settings từ đâu vì bạn chưa đặt DJANGO_SETTINGS_MODULE hoặc gọi configure().
'''
from django.conf import settings
print(settings.DEBUG) 

'''
'''
import os
from django.conf import settings

os.environ["DJANGO_SETTINGS_MODULE"] = "myproject.settings"
settings.configure(DEBUG=True)  # ❌ LỖI! Đã có DJANGO_SETTINGS_MODULE rồi.

'''
'''
from django.conf import settings

print(settings.DEBUG)  # Truy cập settings trước
settings.configure(DEBUG=True)  # ❌ LỖI! settings đã được sử dụng.
```
```table-of-contents
```
# Introduce
Django là một web framework, được thiết kế để có thể implement các web development task một cách nhanh chóng và dễ dàng ➡️ phát triển các database-driven web app.

# Django Project
- Để khởi tạo một ứng dụng django, chúng ta cần auto-generate một số code để thiết lập một Django project
- ***Django project***: là một tập hợp các cài đặt cho một Django instance, các cài đặt này bao gồm ***các cấu hình database, các cấu hình Django và các cấu hình cho application của người dùng***
```bash
# Tạo thư mục djangotutorial
mkdir djangotutorial
# Tạo project vuotnh trong thư mục djangotutorial
django-admin startproject vuotnh djangotutorial
```
* Cấu trúc thư mục của một ***project*** như sau:
```
djangotutorial/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

`manage.py` : Một command-line utility cho phép người dùng tương tác với Django project ([[Django-admin and manage.py]])

`mysite/`: là thư mục chứa các Python package của project, chứa source code logic của project. Tên thư mục chính là tên ***package python*** mà ta cần sử dụng khi import các module bên trong thư mục này. Ví dụ, nếu ta có file ***urls.py*** bên trong ***mysite/***, ta có thể import như sau
```python
from mysite import urls
```

`mysite/__init__.py`: một file rỗng, file này giúp python hiểu folder này là một <span style="color:rgb(255, 0, 0)">Python package</span>
`mysite/settings.py`: Chứa các Settings và configuration của project django ([[Django settings]])
`mysite/urls.py`: định nghĩa các URL trong django project [[URL dispatcher]].
`mysite/asgi.py`: chỉ định entrypoint mà webserver hỗ trợ [ASGI](https://asgi.readthedocs.io/en/latest/) sẽ giao tiếp với django application.
`mysite/wsgi.py`: chỉ định entrypoint mà các webserver hỗ trợ WSGI sử dụng để giao tiếp với django application [[WSGI]]

| WSGI                        | ASGI                                      |
|-----------------------------|------------------------------------------|
| Chỉ hỗ trợ HTTP            | Hỗ trợ HTTP, WebSockets, gRPC           |
| Đồng bộ (synchronous)      | Hỗ trợ bất đồng bộ (asynchronous)       |
| Dùng `wsgi.py`             | Dùng `asgi.py`                          |
| Chạy bằng Gunicorn, mod_wsgi | Chạy bằng Daphne, Uvicorn, Hypercorn    |
# Project và Applications

Khái niệm ***project*** trong Django là một Django web application

Mỗi project Python được định nghĩa chính bởi một `settings module`, ngoài ra còn một vài config khác.

Khi thực thi lệnh tạo ra một project `django-admin startproject mysite`. Một folder `mysite` được tạo ra ➡️ ***project root directory***.
Mỗi project directory sẽ bao gồm
* một file `manage.py`
* thư mục chứa các file cấu hình `settings.py, urls.py, asgi.py, wsgi.py`.

***Application*** trong django được coi như là một `python package`, cung cấp một số features. Ngoài ra ***application*** còn có thể được tái sử dụng ở nhiều project khác.

***Application*** bao gồm `models, views, templates, static files, urls, middleware,...`. Các ***application*** được load vào project thông qua section `INSTALLED_APPS` trong `settings.py` của project, `urls` thì được import vào `urls` của project, tương tự với `middleware`.

## Config applications

- Để cấu hình một application, tạo module `apps.py` trong thư mục của application, sau đó định nghĩa  một app class kế thừa `django.apps.AppConfig` trong đó có `name property`.
- Trong `INSTALLED_APPS`, chỉ cần thêm `name property` của  class app vào trong mảng ➡️ Django sẽ tự động nhận diện `MyAppConfig`
- Nếu muốn Django không tự động chọn class đó để config, ghi đè thuộc tính `AppConfig.default = False` . Khi đó Django sẽ không tự động chọn `MyAppConfig` nữa, và phải chỉ rõ class nào cần dùng trong `INSTALLED_APPS`.
```python
class MyAppConfig(AppConfig):
    name = 'myapp'
    default = False

INSTALLED_APPS = [
    ...,
    "polls.apps.MyAppConfig",
    ...,
]
```

* Nếu có nhiều subclass của `AppConfig` trong file `apps.py`, Django sẽ phải chọn 1 class có `default = True`
* Nếu không thấy subclass nào của AppConfig, Django sẽ dùng class mặc định

# Django khởi động như thế nào ?

Khi Django được khởi động, `django.setup()` chịu trách nhiệm load các component cần thiết để hệ thống được khởi tạo và sẵn sàng hoạt động.

`django.setup(set_prefix=True)` sẽ làm các nhiệm vụ sau:
1. Load các cấu hình từ `settings.py`
2. Nếu có cấu hình logging trong `settings.py`, Django sẽ tạo hệ thống logging theo cấu hình trên.
3. Nếu thiết lập `set_prefix=True` thì Django sẽ lấy giá trị từ `FORCE_SCRIPT_NAME` trong `settings.py`, nếu không nó dùng mặc định `/` ➡️ Sử dụng khi deploy Django với subpath `/myapp/` thay vì `/`.
4. Khởi tạo ***Application Registry*** - là nơi lưu tất cả các ứng dụng có trong `INSTALLED_APPS`.

Django sẽ tự động gọi hàm `django.setup()` trong các trường hợp sau:
* Khi chạy server HTTP (asgi/wsgi) bằng `python manage.py runserver` hoặc deploy app qua Gunicorn/ASGI server.
* Khi chay các lệnh `manage.py` ví dụ `python manage.py migrate`, `python manage.py shell`
Nếu muốn chạy script Python độc lập, không sử dụng `manage.py`, **phải tự gọi django.setup() để sử dụng Django**.

## Application registry hoạt động như thế nào ?

### Giai đoạn 1: Import ứng dụng trong INSTALLED_APPS

* Với mỗi item trong `INSTALLED_APPS`
	* Nếu là chuỗi str trỏ tới class AppConfig (VD: `myapps.app.MyAppConfig`), django sẽ import package gốc của app (theo `name = "myapp" trong AppConfig)
	* Nếu là chuỗi trỏ tới ***tên package*** (VD: `"myapp"`), Django sẽ tìm file `apps.py` trong thư mục `myapp`, nếu có AppConfig trong `apps.py` -> dùng config đó, nếu không tự tạo một `AppConfig` mặc định.
***Không nên import model ở giai đoạn này***
* Các file `apps.py`, `__init__.py` ***không được import model*** dù trực tiếp hay gián tiếp (import thông qua module khác) vì Django chưa load ***model registry*** => Hệ thống có thể bị lỗi

### Giai đoạn 2: Import models

Sau khi cấu hình app trong, Django sẽ tìm và import module `models.py` hoặc thư mục `models` trong mỗi app

➡️ bắt buộc phải định nghĩa toàn bộ các model trong `models.py` hoặc module `models/`

Sau khi import model thì ta có thể sử dụng model
```python
from django.apps import apps
apps.get_model('myapp', 'MyModel')
```

### Giai đoạn 3: Gọi ready() trong AppConfig

Django sẽ lần lượt gọi tới phương thức `ready()` trong class `AppConfig` của mỗi app nếu có overwrite lại phương thức này.

Trong phương thức `ready()` của mỗi app, chúng ta nên:
* Kết nối các `signal handlers`
* Thực hiện các thao tác khởi động cần thiết

***<span style="color: red">Trong các chương trình bình thường, `ready()` method chỉ được gọi  1 lần duy nhất bởi Django lúc khởi động.</span>*** 
Tuy nhiên trong một số trường hợp, `ready` có thể bị gọi nhiều hơn 1 lần.
# Các thành phần chi tiết
[[Django database]]
[[View topic]]

https://docs.djangoproject.com/en/5.2/ref/applications/
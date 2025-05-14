**View function** hay *view* là một python function, nhận vào web request và trả về web response.
- View thường được định nghĩa trong file `views.py` của project hoặc app.

***Mapping URLs to views*** [[URL dispatcher]]

### Async view
Giống như các function bất đồng bộ khác, ***views*** cũng có thể là một hàm bất đồng bộ (async function), hàm này được định nghĩa với cú pháp `async def`
- Khi ta định nghĩa func như trên, Django sẽ tự động phát hiện ra các function này và chạy chúng trong môi trường bất đồng bộ
- Tuy nhiên, khi sử dụng các ***async view***, nên sử dụng *async server* ***ASGI*** để tối ưu hóa hiệu suất
```python
import datetime
from django.http import HttpResponse

async def current_datetime(request):
    now = datetime.datetime.now()
    html = '<html lang="en"><body>It is now %s.</body></html>' % now
    return HttpResponse(html)
```

### View decorators

Nếu quên decorator là gì rồi thì đọc ở đây [[Decorator]]
Django cung cấp một số decorator có thể apply vào các view func để hỗ trợ các tính năng HTTP.

Các decorator được import thông qua `django.views.decorators`. 
***require_http_methods(request_method_list)***
```python
from django.views.decorators.http import require_http_methods
@require_http_methods(["GET", "POST"])
def my_view(request):
    pass
```

Tham khảo các decorator phổ biến thường được sử dụng [ở đây](https://docs.djangoproject.com/en/5.1/topics/http/decorators/)

## Upload file

***Các file khi được upload lên sẽ được lưu ở đâu ?***
Trước khi được django xử lý, các file được upload lên sẽ được lưu theo 2 cách
1. Với các file < 2.5MB, django sẽ lưu nội dung file đó trong memory => giúp cho việc đọc ghi file nhanh
2. Với các file > 2.5MB, Django sẽ ghi uploaded file vào ***temporary file*** được lưu ở thư mục temporary của hệ thống. Ví dụ với hệ thống linux, Django sẽ tạo ra một file có tên giống dạng `/tmp/tmpzfp6I6.upload`.
Có thể thay đổi các Django handler các uploaded file bằng cách thay đổi các cấu hình trong `settings.py` trong [link](https://docs.djangoproject.com/en/5.1/ref/settings/#file-upload-settings)


## Middleware
Middleware thường đứng giữa request và response. Mỗi middleware chịu trách nhiệm thực hiện một số tính năng cụ thể.

### Writing custom middleware
***Middleware factory*** là một lời gọi hàm, nhận lời gọi tới hàm `get_response` làm tham số đầu vào và trả về một `middleware`. `Middle` nhận vào một request và trả về một response
```python
def simple_middleware(get_response):
    # One-time configuration and initialization.
    def middleware(request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.
        response = get_response(request)
        # Code to be executed for each request/response after
        # the view is called.
        return response
    return middleware
```

Ngoài ra có thể định nghĩa middleware dưới dạng một class với func `__call__`
```python
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization.
    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.
        response = self.get_response(request)
        # Code to be executed for each request/response after
        # the view is called.
        return response
```

Hàm `get_response` là lời gọi tới một middleware tiếp theo trong chuỗi các middleware của Django hoặc là lời gọi tới một view function (nếu middleware hiện tại là middleware cuối cùng được cấu hình trong `MIDDLEWARE` của django settings).
Khi middleware hiện tại gọi `get_response`, nó sẽ kích hoạt việc thực thi middleware kế tiếp.

💡***Chú ý***: khi middleware cuối cùng trong chuỗi nhận được `get_response`, thì hàm `get_response` này không phải là một view function, mà nó là một `wrapper method` được cung cấp bởi `request handler` của Django.

❓***Tại sao middleware cuối cùng không trực tiếp gọi đến view?***
Việc middleware cuối cùng không trực tiếp gọi đến view mà thông qua một `wrapper method` giúp đảm bảo tính nhất quán trong quá trình xử lý của Django
1. Middleware có nhiệm vụ xử lý các tác vụ tiền xử lý (trước view) và hậu xử lý (sau view) => giúp cho bản thân view tập trung vào logic nghiệp vụ cụ thể cho một URL
2. Djang có nhiều loại middleware khác nhau, không chỉ middleware xử lý request và response. Có những middleware đặc biệt được thiết kế để ***chạy trước khi view được gọi (process_view)*** và ***sau khi view trả về response (process_template_response, process_exception)***. `wrapper method` đảm bảo các loại middleware này được gọi đúng thời điểm.
3. Request handler chịu trách nhiệm về các tác vụ liên quan đến việc gọi view, ví dụ như việc truyền đúng các arguments từ URL vào view function => nhiệm vụ của wrapper method.

***Luồng thực thi của wrapper method***

Khi ***middleware cuối cùng*** gọi `get_response`, nó sẽ gọi tới `wrapper method`, method đó sẽ lần lượt thực hiện các bước sau
1. Thực thi các view middleware (`process_view`)
	- Ngay trước khi view func được gọi, Django sẽ duyệt qua tất cả các middleware được đăng ký và gọi phương thức `process_view` của chúng (nếu middleware đó có define method này)
	- `process_view(request, view_func, view_args, view_kwargs)`: có `request` là một `HttpRequest` object, `view_func` là python func mà django sử dụng, `view_args` là danh sách các `arguments` được truyền vào trong view, `view_kwargs` là một dict các keyword arguments được truyền vào trong view
	- Nếu kết quả `process_view` của toàn bộ các middleware đều trả về `None` ➡️ Django sẽ gọi tới view
	- Nếu có một `process_view` của middleware nào đó trả về một `HttpResponse` object thì ngay lập tức trả về `HttpResponse` đó và bỏ qua các `process_view`  và `view` tiếp theo.
2. Gọi view function: Nếu toàn bộ các `process_view` đều trả về `None` -> Django sẽ gọi tới view function
3. Thực thi  template-response middleware (`process_template_response`)
	* Nếu view function trả về một object `TemplateResponse`, wrapper method sẽ duyệt qua các middleware và gọi phương thức `process_template_response(request, response)` của chúng.
	* Cuối cùng, response sẽ được render thành một `HttpResponse` hoàn chỉnh.
4. Thực thi `exception middleware` (process_exception)
	* Nếu bất kỳ giai đoạn nào trong quá trình xử lý (từ middleware `process_request`, `process_view` đến việc gọi view) gây ra một exception, Django sẽ kích hoạt cơ chế xử lý lỗi
	* `Wrapper method` sẽ duyệt qua toàn bộ các middleware, gọi phương thức `process_exception(request, exception)` của chúng ***nếu có***.
	* Các middleware này có thể xử lý exception, ghi log, hoặc trả về một `HttpResponse` tùy chỉnh phù hợp với từng yêu cầu.
	* Nếu không có middleware nào xử lý exception, Django sẽ sử dụng cơ chế xử lý lỗi mặc định.
```python
from django.shortcuts import redirect
from django.urls import reverse
class LoginRequiredMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        return self.get_response(request)

    def process_view(self, request, view_func, view_args, view_kwargs):
        # Kiểm tra xem view function có attribute 'login_required' là True hay không
        if hasattr(view_func, 'login_required') and view_func.login_required:
            # Kiểm tra xem người dùng đã đăng nhập hay chưa
            if not request.user.is_authenticated:
                # Nếu chưa đăng nhập, chuyển hướng đến trang đăng nhập
                return redirect(reverse('login'))
        return None  # Nếu không cần chặn, trả về None để Django tiếp tục xử lý
```

### Các kích hoạt middleware
Để sử dụng một middleware nào đó, ta cần đăng ký middleware đó vào `MIDDLEWARE` list trong file `settings.py`. Mỗi middleware được đại diện bởi một `fully python path to the middleware factory 's class` hoặc một `function name`.

Django sẽ áp dụng các middleware theo thứ tự mà chúng được khai báo trong `MIDDLEWARE` ở `settings.py`, `top-down`

### Exception Handler
Django sẽ tự động chuyển đổi các exception thành `HttpResponse` ở giữa các lớp middleware ➡️ các middleware không cần phải bọc lệnh gọi `get_response` trong khối `try/except` để bắt và xử lý các exception.

***ví dụ:*** Ngay cả khi middleware tiếp theo trong chuỗi xử lý gây ra một exception `Http404`, middleware hiện tại sẽ không trực tiếp nhìn thấy exception đó. Thay vào đó, khi `get_response` trả về, middleware hiện tại sẽ nhận được một đối tượng `HttpResponse` với `status_code` là 404, như thể middleware tiếp theo đã trả về một trang "Not Found" thông thường

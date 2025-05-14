Django hỗ trợ việc tạo và quản lý phiên làm việc (***sessions***) cho những người dùng chưa xác thực khi truy cập vào trang web.

## Các đặc điểm của session
***Session framework***: được tích hợp sẵn trong django, chịu trách nhiệm quản lý các phiên làm việc.
* Có thể lưu bất kỳ loại dữ liệu nào trong session của một user truy cập. Sau đó có thể truy xuất dữ liệu này trong các lần truy cập trang web tiếp theo của người dùng trong cùng một phiên
- Mỗi lần user truy cập trang web sẽ có một session mới được tạo ra.
- Dữ liệu trong session được lưu trên server => ko lộ trực tiếp với người dùng
- Django tự động xử lý việc gửi và nhận cookie.
- Khi user truy cập trang web lần đầu tiên (nếu chưa tồn tại session nào), Django sẽ tự động tạo ra một session với ***session_id***. ***session_id*** sẽ được ***lưu trong cookie trên trình duyệt của người dùng***.
- Cookie gửi tới trình duyệt của người dùng chỉ chứa ***session ID***, còn dữ liệu liên quan tới session vẫn được lưu trữ trên server.
- Django cung cấp nhiều cách để lưu dữ liệu của một session. Các phổ biến nhất là lưu trong database. Tuy nhiên có một số cách khác sẽ lưu toàn bộ dữ liệu của session vào trong cookie => có thể bị rò rỉ dữ liệu.
<span style="color: green">session ID thường được lưu ở cookie, vì có thể ngăn chặn Javascript truy cập vào cookie thông qua flag </span> `HttpOnly`

## Session trong Django

Session trong Django được sử dụng như một ***middleware***. Để sử dụng session, ta cần cấu hình thêm class `django.contrib.sessions.middleware.SessionMiddleware` vào trong `MIDDLEWARE` section của `settings.py`.

Để xóa không sử dụng session, xóa `SessionMiddleware` trong `MIDDLEWARE` và `django.contrib.sessions` trong `INSTALLED_APPS`.

## Cấu hình session

Mặc định, Django sẽ lưu sessions trong database thông qua model `django.contrib.sessions.models.Session`.

Tham khảo một số cách cấu hình lưu sessions trong [link](https://docs.djangoproject.com/en/5.2/topics/http/sessions/#configuring-the-session-engine)

## Sử dụng sessions trong views

Khi sử dụng `SessionMiddleware`, mỗi object `HttpRequest` được truyền vào view function sẽ đều có một attribute là `session`.
Có thể đọc hoặc ghi dữ liệu vào session thông qua `request.session` ([ref](https://docs.djangoproject.com/en/5.2/topics/http/sessions/#using-sessions-in-views))


## Clearning the session store

Khi một user login vào hệ thống, Django sẽ thêm một row và bảng `django_session`. Django sẽ cập nhật row này mỗi khi dữ liệu trong session bị thay đổi.
Khi user logout ra khỏi hệ thống, session lưu ở row đó cũng sẽ bị xóa.

Tuy nhiên Django không có cơ chế tự động xóa các session bị hết hạn, do đó nếu người dùng không logout ra khỏi hệ thống -> session sẽ tồn tại mãi mãi.
➡️ Django cung cấp công cụ `clearsessions` để người dùng có thể tự xóa các expired session bằng tay.

## <span style="color: red">Session security</span>

Tấn công cố định phiên (***Session Fixation***)
Khi trang web của bạn có nhiều subdomain ví dụ `blog.example.com`, `shop.example.com`,... trong cùng một trang web `example.com` có khả năng thiết lập cookie trên trình duyệt của người dùng cho toàn bộ domain gốc `*.example.com`.
➡️ Một cookie được thiết lập bởi 1 subdomain có thể được gửi đến tất cả các domain khác thuộc cùng domain chính.

<span style="color: orange">Ví dụ</span>
*  hacker đăng nhập vào một subdomain hợp lệ `good.example.com`, nhận được một session hợp lệ cho tài khoản của hacker => gọi là ***hacked_session_id***
* Hacker sở hữu một máy chủ có domain `bad.example.com`, hacker sử dụng máy chủ này giả mạo máy chủ thật gửi ***hacked_session_id*** về cho người dùng. 
* Trình duyệt của người dùng sẽ lưu lại ***hacked_session_id*** này trên cookie, do một subdomain (`bad.example.com`) được phép thiết lập cookie cho toàn bộ các domain (`*.example.com`).
* Khi victim truy cập các subdomain khác ví dụ như `good.example.com`, trình duyệt sẽ gửi cookie chứa ***hacked_session_id*** lên server ➡️ victim đang đăng nhập với session của hacker ➡️ nếu vô tình session lưu lại các dữ liệu nhạy cảm => dữ liệu bị lưu ở session của hacker.
![[Pasted image 20250409230311.png]]

***Cách fix lỗ hổng này***
* Thay đổi session ID ngày sau khi người dùng login ➡️ loại bỏ hầu hết các lỗ hổng session fixation
* Chỉ chấp nhận session từ cookie, không chấp nhận session ID từ HTTP request hoặc HTTP header (client không thể chỉnh sửa cookie trên trình duyệt).
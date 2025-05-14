***URLconf (URL configuration)*** là một module Python giúp ***định tuyến các URL đến các views trong Django***.
Là tập hơn các quy tắc giúp Django biết khi người dùng truy cập một URL thì nên gọi view nào để xử lý

## Luồng Django xử lý một request
1. **Xác định `ROOT_URLCONF`***
	- Khi có một request đến, Django sẽ phải xác định ***module URL gốc*** cần sử dụng
	- Mặc định thì django sẽ lấy từ settting `ROOT_URLCONF` trong `settings.py`, tuy nhiên nếu request có thuộc tính `urlconf` (được config bởi middleware), Django sẽ ưu tiên sử dụng giá trị đó thay vì `ROOT_URLCONF`.

2. **Load module python chứa**  `urlpatterns`
	* Django sẽ import python module được chỉ định bởi value của `ROOT_URLCONF`.
	* Trong module này, Django sẽ tìm kiếm biến có tên `urlpatterns`. Biến này gồm một list các instance của `django.urls.path()` hoặc `django.urls.re_path()`.
		* `django.urls.path()`: dùng để định nghĩa các URL patterns đơn giản, dựa trên đường dẫn ([tham khảo](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.path))
		* `django.urls.re_path()`: dùng để định nghĩa các URL patterns phức tạo, sử dụng regex để xác định ([tham khảo](https://docs.djangoproject.com/en/5.1/ref/urls/#django.urls.path))
3. **Duyệt qua các URL pattern**
	* Django so sánh `path_info` của request với từng URL pattern trong danh sách `urlpatterns` theo thứ tự, và dừng lại ở URL pattern đầu tiên khớp với `path_info`.
4. **Gọi view xử lý**
	* khi tìm dc URL pattern phù hợp, Django sẽ import và gọi view tương ứng. View là một `function-base view` hoặc `class-based view` được chỉ định trong URL pattern
	* View sẽ được truyền các đối số sau:
		* một instance của class `HttpRequest`
		* các đối số khác được chỉ định trong `kwargs` của `django.urls.path()` hoặc `django.urls.re_path()`.
5. **Xử lý lỗi***
	* Nếu ko có URL pattern nào phù hợp, Django sẽ gọi view xử lý lỗi 404 hoặc bắn ra exception tùy chỉnh



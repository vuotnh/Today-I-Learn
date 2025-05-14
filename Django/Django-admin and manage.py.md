***django-admin*** là một công cụ dòng lệnh của django dùng để quản lý các administrative task như chạy server, thực hiện migration, tạo superuser,....

***manage.py*** là file được tạo tự động khi tạo project django, file này có chắc năng tương tự như ***django-admin*** tuy nhiên khác ở chỗ ***manage.py*** sẽ tự động đặt biến môi trường `DJANGO_SETTINGS_MODULE` để trỏ đến file ***settings.py*** của dự án.

Nếu sử dụng ***django-admin*** ta phải chỉ định biến môi trường thủ công để trỏ đến file ***setttings.py***
```bash
DJANGO_SETTINGS_MODULE=myproject.settings django-admin runserver
```

Tham khảo thêm các option ở đây: https://docs.djangoproject.com/en/5.1/ref/django-admin/
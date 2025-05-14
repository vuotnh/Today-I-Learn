[[Django model]]

## Tạo index với django mà không có downtime

Khi database được đưa vào hoạt động, ta quan sát thấy  việc query xuống database bị chậm ở một số cột ➡️ Do dữ liệu quá lớn, nên cần tạo `index` trong database để tối ưu query.

Các thông thường, tạo `makemigrations` sau đó migrate vào database
```python
# models.py

from django.db import models

class Sale(models.Model):
    sold_at = models.DateTimeField(
        auto_now_add=True,
        db_index=True,
    )
    charged_amount = models.PositiveIntegerField()
```

```bash
python manage.py makemigrations
python manage.py migrate
```

<span style="color: red; font-style: italic; font-weight: bold">Tuy nhiên với cách làm này, khi Django tạo index trong table, nó sẽ <strong>lock table</strong> cho tới khi việc tạo index thành công</span> ⇒ <span style="color: green; font-style: italic; font-weight: bold">nếu bảng có size quá lớn thì sẽ gây downtime</span>

https://realpython.com/create-django-index-without-downtime/

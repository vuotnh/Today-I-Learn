```table-of-contents
```
Model là nơi định nghĩa thông tin về bảng database trong django. Model bao gồm các trường cần thiết, cách django lưu dữ liệu. Mỗi model chỉ map tương ứng với 1 table trong database
* Mỗi model là một python class, kế thừa class `django.db.models.Model`.
* Mỗi thuộc tính của class đại diện cho một field trong row
```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

## Sử dụng model

Sau khi định nghĩa một model, cần cấu hình cho Django sử dụng model đó bằng cách ***thêm tên của app có chưa file models.py*** vào section `INSTALLED_APPS` trong `settings.py`.
Ví dụ models `Student` được đinh nghĩ trong file `models.py` thuộc app `room` nằm trong module `school`:
```python
INSTALLED_APPS = [
	# ...
	"school.room",
	# ...
]
```
Sau khi thêm một app mới vào `INSTALLED_APPS`, cần chạy lại `python manage.py makemigrations` và `python manage.py migrate` để gen ra các file migration cho model và ghi chúng vào database.

Tham khảo việc định nghĩa các field trong model ở [đây](https://docs.djangoproject.com/en/5.2/topics/db/models/#fields)

## Relationships

### Many-to-one
Để định nghĩa quan hệ `many-to-one`, ta sử dụng trường trường `django.db.models.ForeignKey` trong model
Ví dụ: model `Car` có một `Manufacturer`, tuy nhiên một `Manufacturer` lại có nhiều `Car`
```python 
from django.db import models

class Manufacturer(models.Model):
    pass
# chỉ định ForeignKey ở bảng con
class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
```

### Many-to-many
Thường sẽ định nghĩa một bảng trung gian cho quan hệ `ManyToManyField`
Ví dụ
```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through="Membership")

    def __str__(self):
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)

    class Meta:
        constraints = [
            models.UniqueConstraint(
                fields=["person", "group"], name="unique_person_group"
            )
        ]
```
Bảng trung gian `intermediary model` được liên kết với hai bảng thông qua `through`
[Tham khảo chi tiết](https://docs.djangoproject.com/en/5.1/topics/db/models/#extra-fields-on-many-to-many-relationships)


### OneToOneField
[Tham khảo](https://docs.djangoproject.com/en/5.1/topics/db/models/#one-to-one-relationships)

### Meta options

Metadata của một model được định nghĩa thông qua một `inner class Meta`
```python
from django.db import models
class Ox(models.Model):
    horn_length = models.IntegerField()
    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```
Model metadata sẽ định nghĩa ***anything that 's not a field*** ví dụ như ordering options (`ordering`), database table name (`db_table`),.....
Việc thêm `class Meta` vào model là ***không bắt buộc***.

### Model method
Ta có thể định nghĩa các custom method hoặc overwrite các method có sẵn của model class
[Tham khảo](https://docs.djangoproject.com/en/5.1/topics/db/models/#model-methods)

### Abstract base classes
Abstract base classes được sử dụng khi muốn định nghĩa một số common information vào các model implement abstract đó.
Abstract model là model bình thường, tuy nhiên có thêm thuộc tính `abstract=True` vào `Meta` class.
```python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

###  Related name trong model

Khi bạn có một model A liên kết đến model B (ví dụ qua `ForeignKey`), thì từ model B bạn có thể truy cập tất cả các đối tượng A liên kết đến nó **thông qua `related_name`**.
```python
# Ví dụ trong many to one
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')

# sử dụng related_name
author = Author.objects.get(id=1)
books = author.books.all()  # 'books' là giá trị của related_name

# Ví dụ trong one to one
class User(models.Model):
    username = models.CharField(max_length=100)

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')

user = User.objects.get(id=1)
user.profile  # truy cập qua related_name
```

### Proxy model

Một **proxy model** là một model **dựa trên một model khác**, **không tạo bảng mới**, **không có thêm field nào mới**, nhưng có thể có **hành vi khác** trong code Python.

Bản chất của proxy model chỉ là logic trên python, không có liên hệ gì dưới database.

Cách khai báo một proxy model:
```python
class Person(models.Model):
    name = models.CharField(max_length=100)
    birthdate = models.DateField()

    def say_hello(self):
        return f"Hello, I am {self.name}"
```
Giờ ta muốn có một "view" khác của `Person`, ví dụ `Employee`, có cách sắp xếp khác hoặc thêm method mới.
```python
class Employee(Person): # model mới kế thừa lại model ban đầu
    class Meta:
        proxy = True  # ⚠️ Đánh dấu đây là Proxy model
        ordering = ['birthdate']  # Thay đổi cách sắp xếp mặc định

    def is_birthday(self):
        from datetime import date
        return self.birthdate.month == date.today().month and self.birthdate.day == date.today().day

```

Sử dụng proxy model
```python
e = Employee.objects.first()
print(e.name)  # dữ liệu của e vẫn được lấy từ bảng 'person'
# tuy nhiên hành vi của model proxy khác với model ban đầu
print(e.is_birthday())  # method mới từ proxy
```
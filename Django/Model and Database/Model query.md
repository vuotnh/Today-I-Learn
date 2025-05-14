```table-of-contents
```
# Model query
Khi môt data models được tạo, Django cho phép dev sử dụng các `database-abstraction API` để tương tác với database (create, retrieve, update, delete).

Mỗi model class là một database table ↔️ mỗi instance của class là một row trong table đó.

```python
from blog.models import Blog
blog_row = Blog(name="vuotnh", content="Hello world")
blog_row.save()
```

## QuerySet

Khi dùng database model và gọi các hàm như `.filter()`,  `.all()` hoặc <span style="color: green">các hàm query dữ liệu khác</span> thì sẽ có kết quả trả về là một `QuerySet`
 ***QuerySet là một danh sách các đối tượng được lấy ra từ trong database thông qua truy vấn nào đó.***

QuerySet có tính `lazy` vì nó không truy vấn vào database để lấy dữ liệu ngay khi được call mà chỉ đợi đến khi ***thực sự cần dùng dữ liệu*** với thực hiện truy vấn ➡️ `lazy evaluation`
```python
# Ở đây vẫn chưa thực hiện truy vấn, books ở đây vẫn là QuerySet chứ chưa có dữ liệu
books = Book.objects.filter(author="Alice")

# Lúc này cần dùng dữ liệu
# Django sẽ query vào database, lấy kết quả trả về, map thành các object python và lưu vào variable trong RAM
for book in books:
    print(book.title)
print(list(books))
```

Khi sử dụng nhiều hàm query, mỗi bước sẽ tạo ra một QuerySet với với điều kiện mới (điều kiện mới sẽ dựa trên điều kiện trước đó)
```python
books = Book.objects.filter(author="Alice").exclude(title="Secret").order_by("title")
# filter(...): Lấy sách của Alice 
# exclude(...): Bỏ sách có tên là "Secret"
# order_by(...): Sắp xếp theo tiêu đề
```

*Các thao tác như `update()` hay `delete()` là các method của QuerySet, chúng thực hiện luôn thao tác thay đổi dữ liệu dưới database chứ không trả về QuerySet mới*

## Manager

Manager là trình quản lý truy vấn mặc định cho model trong Django
Manager là cầu nối giữa model và QuerySet.
Khi viết `Book.objects.all()` =>  `objects` chính là Manager
* `objects` là một Manager
* `.all()` là phương thức truy vấn được gọi từ Manager
* Kết quả trả về là một `QuerySet` chứa dữ liệu tất cả các Book

### Custom Manager
```python
class BookManager(models.Manager):
    def published(self):
        return self.filter(is_published=True)

# Dùng manager trong model
class Book(models.Model):
    title = models.CharField(max_length=100)
    is_published = models.BooleanField(default=False)

    objects = models.Manager()  # Default manager
    publish_manager = BookManager() # custom Manager

# Sử dụng các manager và custom manager
Book.objects.all() 
Book.publish_manager.published()
```

# Performing raw SQL queries

Để thực thi các raw SQL queries trong Django, có thể sử dụng 2 cách:
1. dùng `Manager.raw()` để xử lý raw queries và trả về một model instances
2. Thực thi trực tiếp câu lệnh SQL

## Cách 1: Sử dụng Manager.raw()

Method `raw()` của manager được sử dụng để thực thi các raw SQL queries và trả về một model instances.
`Manager.raw(raw_query, params=(), translations=None)`

Method trên nhận vào một raw SQL query, thực thi chúng và trả về một instance của `django.db.models.query.RawQuerySet`.
```python
class Person(models.Model):
    first_name = models.CharField(...)
    last_name = models.CharField(...)
    birth_date = models.DateField(...)

for p in Person.objects.raw("SELECT * FROM myapp_person"):
	print(p)
```

> `raw()` không kiểm tra nội dung câu lệnh mà người dùng truyền vào, dev có thể truyền vào bất cứ câu lệnh (SELECT, UPDATE, DELETE, INSERT,...) nào miễn nó là chuỗi string hợp lệ.

[Tham khảo chi tiết thêm] (https://docs.djangoproject.com/en/5.1/topics/db/sql/#performing-raw-queries)

## Sử dụng trực tiếp lệnh SQL

Trong Django, đối tượng `django.db.connection` đại diện cho database connection mặc định. Để sử dụng database connection, ta cần gọi tới `connection.cursor()` để lấy `cursor object`. Sau đó call tới `cursor.execute(sql, [params])` để thực thi SQL. Dùng `cursor.fetchone()` hoặc `cursor.fetchall()` để trả về rows.

```python
from django.db import connection

def my_custom_sql(self):
	with connection.cursor() as cursor:
		cursor.execute("UPDATE bar SET foo = 1 WHERE baz = %s", [self.baz])
        cursor.execute("SELECT foo FROM bar WHERE baz = %s", [self.baz])
        row = cursor.fetchone()
    return row

def my_custom_sql_2(self):
	c = connection.cursor()
	try:
		c.execute("UPDATE bar SET foo = 1 WHERE baz = %s", [self.baz])
	finally:
		c.close()
```

Nếu bạn dùng nhiều database, có thể dùng như sau
```python
from django.db import connections

with connections["my_db_alias"].cursor() as cursor:
	......
```

Mặc định thì Python DB API sẽ trả về các kết quả mà không có field names cho từng trường => trả về một `list` value.

## Stored procedures

Sử dụng `CursorWrapper.callproc(procname, params=None, kparams=None` để gọi tới một `database stored procedure` có tên là `procname`.

Ví dụ trong Oracle database có một procedure
```sql
CREATE PROCEDURE "TEST_PROCEDURE"(v_i INTEGER, v_text NVARCHAR2(10)) AS
    p_i INTEGER;
    p_text NVARCHAR2(10);
BEGIN
    p_i := v_i;
    p_text := v_text;
END;
```

sử dụng connection để gọi tới procedure này
```python 
with connection.cursor() as cursor:
	cursor.callproc("test_procedure", [1, "test"])
```
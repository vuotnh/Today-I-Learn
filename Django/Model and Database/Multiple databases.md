```table-of-contents
```

# Khai báo các database

Django hỗ trợ sử dụng nhiều database trong một project.

Các kết nối đến từng loại database sẽ được cấu hình ở list `DATABASES` trong `settings.py`.
[Các thuộc tính để cấu hình database trong DATABASES settings.py](https://docs.djangoproject.com/en/5.1/ref/settings/#std-setting-DATABASES)

```python
DATABASES = {
    "default": {
        "NAME": "app_data",
        "ENGINE": "django.db.backends.postgresql",
        "USER": "postgres_user",
        "PASSWORD": "s3krit",
    },
    "users": {
        "NAME": "user_data",
        "ENGINE": "django.db.backends.mysql",
        "USER": "mysql_user",
        "PASSWORD": "priv4te",
    },
    "customers": {
        "NAME": "customer_data",
        "ENGINE": "django.db.backends.mysql",
        "USER": "mysql_cust",
        "PASSWORD": "veryPriv@ate",
    },
}
```

### Chỉ định database khi migrate
```bash
python manage.py migrate # migrate vào database default
python manage.py migrate --database=users # migrate vào database users
```

# Database routing

Cách dễ nhất để sử dụng multiple databases trong Django đó là tạo một cấu trúc Database routing.
Default routing chắc chắn rằng nếu database không được chỉ định khi thực thi query, Django sẽ sử dụng dajngo mặc định.

## Database routers

Database router là một class có 4 method sau:

`db_for_read(model, **hints)`
Suggest database được sử dụng cho các `read operations` khi sử dụng multiple database
Trả về `None` nếu không có suggestion

`db_for_write(model, **hints)`
Suggest database được sử dụng cho các `write operations` khi sử dụng multiple database
Trả về `None` nếu không có suggestion.

`allow_relation(obj1, obj2, **hints)`
Phương thức này dùng để kiểm tra xem có cho phép tạo relation giữa hai object trong database hay không. Ví dụ như `ForeignKey` hoặc `ManyToMany`.
`obj1`, `obj2` là hai model instance mà Django đang cố tạo quan hệ giữa chúng.
- `True`: Cho phép tạo quan hệ giữa `obj1` và `obj2` (dù chúng có thể nằm ở 2 database khác nhau).
- `False`: **Không cho phép** tạo quan hệ giữa chúng.

`allow_migrate(db, app_label, model_name=None, **hints)`
Dùng để kiểm soát ***Django sẽ migrate model lên database nào*** khi chạy lệnh `pythn manage.py migrate`.
Giúp Django quyết định có nên chạy migration này lên database `db` hay không
* `db`: Tên alias của database
* `app_label`: Tên của app chứa model đang được migrate
* `model_name`: Tên model (viết thường)
Trả về `True` nếu cho phép thực hiện migration này trên database `db`, ngược lại trả về `False`.

## Cách sử dụng Database router

Các database router được cài đặt thông qua `DATABASE_ROUTERS` trong `settings`.
Section `DATABASE_ROUTERS` định nghĩa một list các class name mà Dajngo sẽ sử dụng để quyết định 
* Database nào dùng để đọc/ghi
* Có cho phép tạo quan hệ giữa các model hay không
* Model nào nên migrate vào database nào
```python
DATABASE_ROUTERS = ['myapp.routers.MyRouter]
```

Khi routing, Django sẽ gọi lần lượt từng router ***theo thứ tự trong danh sách định nghĩa*** và sẽ ***dừng ngay khi một router trả về giá trị khác `None`*** và không quan tâm tới các router phía sau.

Ví dụ
```python
DATABASES = {
    "default": {},
    "auth_db": {
        "NAME": "auth_db_name",
        "ENGINE": "django.db.backends.mysql",
        "USER": "mysql_user",
        "PASSWORD": "swordfish",
    },
    "primary": {
        "NAME": "primary_name",
        "ENGINE": "django.db.backends.mysql",
        "USER": "mysql_user",
        "PASSWORD": "spam",
    },
    "replica1": {
        "NAME": "replica1_name",
        "ENGINE": "django.db.backends.mysql",
        "USER": "mysql_user",
        "PASSWORD": "eggs",
    },
    "replica2": {
        "NAME": "replica2_name",
        "ENGINE": "django.db.backends.mysql",
        "USER": "mysql_user",
        "PASSWORD": "bacon",
    },
}
```

```python
DATABASE_ROUTERS = ["path.to.AuthRouter", "path.to.PrimaryReplicaRouter"]
```

```python
import random
class AuthRouter:
    """
    A router to control all database operations on models in the
    auth and contenttypes applications.
    """

    route_app_labels = {"auth", "contenttypes"}

    def db_for_read(self, model, **hints):
        """
        Attempts to read auth and contenttypes models go to auth_db.
        """
        if model._meta.app_label in self.route_app_labels:
            return "auth_db"
        return None

    def db_for_write(self, model, **hints):
        """
        Attempts to write auth and contenttypes models go to auth_db.
        """
        if model._meta.app_label in self.route_app_labels:
            return "auth_db"
        return None

    def allow_relation(self, obj1, obj2, **hints):
        """
        Allow relations if a model in the auth or contenttypes apps is
        involved.
        """
        if (
            obj1._meta.app_label in self.route_app_labels
            or obj2._meta.app_label in self.route_app_labels
        ):
            return True
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        Make sure the auth and contenttypes apps only appear in the
        'auth_db' database.
        """
        if app_label in self.route_app_labels:
            return db == "auth_db"
        return None

class PrimaryReplicaRouter:
    def db_for_read(self, model, **hints):
        """
        Reads go to a randomly-chosen replica.
        """
        return random.choice(["replica1", "replica2"])

    def db_for_write(self, model, **hints):
        """
        Writes always go to primary.
        """
        return "primary"

    def allow_relation(self, obj1, obj2, **hints):
        """
        Relations between objects are allowed if both objects are
        in the primary/replica pool.
        """
        db_set = {"primary", "replica1", "replica2"}
        if obj1._state.db in db_set and obj2._state.db in db_set:
            return True
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        All non-auth models end up in this pool.
        """
        return True
```

# Manually selecting database

### Manually selecting a database for a QuerySet
```python
Author.objects.all()
Author.objects.using("default")
Author.objects.using("other")
```

### Select a database for save()

```python
my_object.save(using="user_table")
```

### Move an object from one database to another

```python
p = Person(name="Fred")
p.save(using="first") # step 1
p.save(using="second") # step 2
```

Sau khi thực hiện xong step 1, p lúc này đã có primary key, khi thực hiện step 2, nó sẽ ghi p vào database 2
* Nếu primary key chưa có trong database 2, nó sẽ tự động copy
* Nếu primary key đã có trong database 2, nó sẽ overwrite dữ liệu đang có trong db 2

### Select database to delete

```python
user = User.objects.using("legacy_users").get(username="vuotnh")
user.delete()

user_obj.delete(using="legacy_users")
```


# Multiple database for Django admin

Mặc định, Django admin không hỗ trợ multiple databases. Do đó để sử dụng multiple database, cần implement một custom `admin.ModelAdmin` class với từng app

```python
class MultiDBModelAdmin(admin.ModelAdmin):
    # A handy constant for the name of the alternate database.
    using = "other"

    def save_model(self, request, obj, form, change):
        # Tell Django to save objects to the 'other' database.
        obj.save(using=self.using)

    def delete_model(self, request, obj):
        # Tell Django to delete objects from the 'other' database
        obj.delete(using=self.using)

    def get_queryset(self, request):
        # Tell Django to look for objects on the 'other' database.
        return super().get_queryset(request).using(self.using)

    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        # Tell Django to populate ForeignKey widgets using a query
        # on the 'other' database.
        return super().formfield_for_foreignkey(
            db_field, request, using=self.using, **kwargs
        )

    def formfield_for_manytomany(self, db_field, request, **kwargs):
        # Tell Django to populate ManyToMany widgets using a query
        # on the 'other' database.
        return super().formfield_for_manytomany(
            db_field, request, using=self.using, **kwargs
        )
```

Các model khác đều được kế thừa từ `MultiDBModelAdmin`


# Hạn chế của việc sử dụng multiple database

## Cross-database relations

Django không hỗ trợ `ForeignKey` hay `ManyToMany` relations ship giữa các multiple databases.
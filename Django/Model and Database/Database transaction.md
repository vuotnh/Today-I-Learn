```table-of-contents
```
# Quản lý các database transaction

Mặc định, Django sẽ hoạt động ở `autocommit mode`, mọi ***query sẽ được ngay lập tức commit xuống database***.

Djang sẽ tự động sử dụng các `transactions` hoặc các `savepoints` mà không cần cấu hình thủ công. 
* `transaction`: nếu có lỗi xảy ra, transaction sẽ rollback toàn bộ các thay đổi được bọc trong transaction
* `savepoint`: một checkpoint tạm thời trong transaction, cho phép rollback về một thời điểm checkpoint thay vì rollback toàn bộ.
➡️ Không cần lo về việc thực thi query bị lỗi gây hỏng dữ liệu.

## Transaction với HTTP request

Có một số cách thông dụng để handle transaction đó là bọc một request trong một transaction.
Để thực hiện việc này , cấu hình `ATOMIC_REQUESTS=True` trong configuration của mỗi database.

Trước khi gọi tới một view function, Django sẽ tự động start một transaction. Nếu view trả về response bình thường, ko có lỗi thì Django sẽ commit transaction đó, ngược lại thì sẽ rollback transactions.

Ngoài ra có thể tạo ra các subtransaction để sử dụng `savepoint` trong view bằng cách sử dụng `transaction.atomic()` context manager.

### Atomic

`atomic` cho phép tạo một block code.
* nếu toàn bộ code trong block chạy ổn, Django sẽ commit xuống DB
* Nếu có `exception` được raise trong block `atomic`, Django sẽ rollback mọi thay đổi bên trong
Ví dụ sử dụng `atomic()`
```python
from django.db import transaction

@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()

def viewfunc_wrap(request):
    # This code executes in autocommit mode (Django's default).
    do_stuff()

    with transaction.atomic():
        # This code executes inside a transaction.
        do_more_stuff()
```

```python
from django.db import IntegrityError, transaction

@transaction.atomic
def viewfunc(request):
    create_parent()

    try:
        with transaction.atomic():
            generate_relationships()
    except IntegrityError:
        handle_exception()

    add_children()
```

<span style="color: red">Chú ý:</span> nên `try except` ***bên ngoài khối*** `atomic`, vì nếu `try except` bên trong khối `atomic` mà sau đó ko `raise exception` nào cả thì `atomic` sẽ hiểu rằng ***block code đó không có lỗi, và sẽ commit xuống database***.

### Performing actions after commit

Để thực hiện một action khi transaction được commit thành công, sử dụng `on_commit()`

`on_commit()` cho phép đăng ký một `callback function`, `callback function` này được thực thi ngay sau khi  transaction hiện tại được commit thành công.

Ví dụ:
```python 
from django.db import transaction

def send_welcome_email():
	print("send mail")

transaction.on_commit(send_welcome_email)
```

Callback function không cho phép truyền argument, tuy nhiên ta có thể sử dụng `partial()` hoặc lamda function để truyền 
```python
from functools import partial
from django.db import transaction

def send_welcome_email(user):
	print(f"send mail to {user}")

transaction.on_commit(partial(send_welcome_email, user=user))
transaction.on_commit(lambda: send_welcome_email(user))
```

<span style="color: green">Mỗi khi on_commit() được gọi, nó sẽ tự động đăng ký một savepoint</span>
<span style="color: red">Nếu transaction thất bại, hoặc raise exception, thì callback sẽ không được gọi.</span>

```python
with transaction.atomic():  # Outer atomic, start a new transaction
    transaction.on_commit(foo)
    try:
        with transaction.atomic():  # Inner atomic block, create a savepoint
            transaction.on_commit(bar)
            raise SomeError()  # Raising an exception - abort the savepoint
    except SomeError:
        pass

# foo() will be called, but not bar()
```

***Savepoint chỉ khả dụng cho các loại database sử dụng InnoDB storage engine như SQLite, PostgreSQL, Oracle và MySQL***

```python
from django.db import transaction

# open a transaction
@transaction.atomic
def viewfunc(request):
    a.save()
    # transaction now contains a.save()
    sid = transaction.savepoint()
    b.save()
    # transaction now contains a.save() and b.save()
    if want_to_keep_b:
        transaction.savepoint_commit(sid)
        # open transaction still contains a.save() and b.save()
    else:
        transaction.savepoint_rollback(sid)
        # open transaction now contains only a.save()
```